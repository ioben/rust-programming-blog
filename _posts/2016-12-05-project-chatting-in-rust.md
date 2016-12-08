---
layout: post
title: "Project: Chatting in Rust"
date: 2016-01-01 12:07:00
---

For our project we created a small chat client and server:

### Configuration files
__config.toml:__
```toml
[package]
name = "lw2-project"
version = "0.1.0"

[dependencies]
log = "0.3.6"
simplelog = "0.4.2"
getopts = "0.2"
```

### Server
```rust
#[macro_use] extern crate log;
extern crate simplelog;
extern crate uuid;

use simplelog::{TermLogger, Config, LogLevelFilter};
use std::collections::HashMap;
use std::net::{TcpListener, TcpStream};
use std::fmt::{self, Display, Formatter};
use std::io::{Write, BufReader, BufRead};
use uuid::Uuid;

use std::thread;
use std::sync::mpsc::{channel, Sender, Receiver};
use std::sync::{Arc, Mutex};

#[derive(Debug, Clone)]
enum ClientMessage {
    Msg(String),
    MetaMessage(String),
    Quit
}

impl Display for ClientMessage {
    fn fmt(&self, f: &mut Formatter) -> Result<(), fmt::Error> {
        match self {
            &ClientMessage::Msg(ref str) => write!(f, "{}", str),
            &ClientMessage::MetaMessage(ref str) => write!(f, "# {}", str),
            &ClientMessage::Quit => write!(f, ""),
        }
    }
}

fn handle_client(channel: Receiver<ClientMessage>, chat_broadcast: Sender<ClientMessage>, stream: TcpStream) {
    let ipaddr = match stream.peer_addr() {
        Ok(ip) => ip,
        Err(e) => {
            error!("Failed to get client's IP address: {:?}", e);
            return;
        }
    };

    info!("Client connected: '{}'", ipaddr);

    chat_broadcast.send(ClientMessage::MetaMessage(format!("Client connected {}", ipaddr)))
        .unwrap();

    // Pass messages to client
    {
        let mut stream = stream.try_clone().unwrap();

        thread::spawn(move || {
            loop {
                match channel.recv().expect(&format!("Failed to fetch messages for client: {}", ipaddr)) {
                    ClientMessage::Msg(msg) => { write!(stream, "{}\n", msg).unwrap(); },
                    ClientMessage::MetaMessage(msg) => { write!(stream, "## {}\n", msg).unwrap(); },
                    ClientMessage::Quit => { break; },
                };
            }
        });
    }

    // Read lines and broadcast them
    let mut reader = BufReader::new(stream);
    let mut buffer = String::new();

    loop {
        match reader.read_line(&mut buffer) {
            Ok(0) => break, // Client disconnected
            Ok(_) => { chat_broadcast.send(ClientMessage::Msg(buffer.trim().to_owned())).unwrap(); },
            Err(e) => { warn!("Client {}: Failed to read line from TCP socket: {}", ipaddr, e) },
        };

        buffer.clear();
    }

    chat_broadcast.send(ClientMessage::MetaMessage(format!("User {} disconnected", ipaddr))).unwrap();
    info!("Client {}: Disconnecting.", ipaddr);
}

// Broadcast messages to other clients
fn handle_broadcasts(clients: Arc<Mutex<HashMap<Uuid, Sender<ClientMessage>>>>, chat_broadcast: Receiver<ClientMessage>) {
    loop {
        let message = match chat_broadcast.recv() {
            Ok(msg) => msg,
            Err(e) => {
                error!("Error broadcasting message to all users: {:?}", e);
                continue;
            },
        };

        let clients = match clients.lock() {
            Ok(clients) => clients,
            Err(e) => {
                error!("Unable to lock client list for message broadcast: {}", e);
                continue;
            }
        };

        info!("Sending {}", message);

        for ref client in clients.values() {
            match client.send(message.clone()) {
                Ok(()) => { },
                Err(e) => { warn!("Failed to send message to user: {}", e); },
            }
        }
    }
}

fn main () {
    // Configure logger
    TermLogger::init(LogLevelFilter::Info, Config::default()).unwrap();

    // Configure port for server to listen on
    let bind_addr = "127.0.0.1:6000";
    let listener = TcpListener::bind(bind_addr)
        .expect(&format!("Failed to bind to {}.", bind_addr));

    info!("Listening on {}", bind_addr);

    // Stores a writable command channel to each client connection thread
    let clients = Arc::new(Mutex::new(HashMap::new()));

    // Broadcast channel, anything written to it will be sent to every client
    let (chat_tx, chat_rx) = channel();

    // Handle broadcasts from current users
    let message_exchange = {
        let clients = clients.clone();
        thread::spawn(move || {
            handle_broadcasts(clients, chat_rx);
        })
    };

    // Handle new incoming connections
    for stream in listener.incoming() {
        match stream {
            Ok(stream) => {
                let (client_tx, client_rx) = channel();
                let client_chat_tx = chat_tx.clone();

                let client_uuid = Uuid::new_v4();

                {
                    let clients = clients.clone();
                    thread::spawn(move || {
                        handle_client(client_rx, client_chat_tx, stream);

                        match clients.lock().unwrap().remove(&client_uuid) {
                            Some(client_tx) => { client_tx.send(ClientMessage::Quit).unwrap(); },
                            None => { },
                        };
                    });
                }

                match clients.lock() {
                    Ok(mut clients) => { clients.insert(client_uuid, client_tx); },
                    Err(e) => { warn!("Failed to lock clients list to add new client: {}", e); },
                };
            },
            Err(_) => { },
        }
    }

    let _ = message_exchange.join();

    // Shutdown
    for client in clients.lock().unwrap().values() {
        match client.send(ClientMessage::Quit) {
            Ok(_) => { },
            Err(e) => { warn!("Failed to quit client thread: {}", e); },
        };
    }
}
```


### Client
```rust
#[macro_use] extern crate log;
extern crate simplelog;
extern crate getopts;


use std::io;
use std::io::prelude::*;
use std::env;
use getopts::Options;
use std::net::{TcpStream, SocketAddr};
use simplelog::{TermLogger, Config, LogLevelFilter};
use std::io::BufReader;
use std::str::FromStr;
use std::thread;

fn main() {
    // Configure logger
    TermLogger::init(LogLevelFilter::Info, Config::default()).unwrap();

    // Parse arguments
    let mut opts = Options::new();
    opts.optopt("s", "server", "Server to connect to", "SERVER");
    opts.optflag("h", "help", "print this help menu");

    let args: Vec<String> = env::args().collect();
    let matches = opts.parse(&args[1..]).unwrap();

    // Configure port for server to listen on
    let server_addr = matches.opt_str("server").unwrap_or(String::from("127.0.0.1:6000"));
    let server_addr = SocketAddr::from_str(&server_addr).expect("Failed to parse given server address");

    info!("Connecting to {}", server_addr);

    let mut stream = TcpStream::connect(server_addr)
        .expect(&format!("Failed to connect to {}.", server_addr));

    println!("Connected!");

    // Other user's messages
    let receiver_thread = {
        let stream = match stream.try_clone() {
            Ok(stream) => stream,
            Err(e) => { panic!(format!("Failed to create message receiving thread: {}", e)); }
        };

        thread::spawn(move || {
            BufReader::new(stream)
                .lines()
                .map(|l| l.expect("Failed to read line from server"))
                .map(|l| println!("> {}", l))
                .last();
        })
    };

    // Communications REPL

    let reader = io::stdin();
    loop {
        let mut buffer = String::new();
        match reader.read_line(&mut buffer) {
            Ok(0) => { break; },
            Ok(_) => { stream.write(&buffer.into_bytes()).expect("Failed to send message"); },
            Err(e) => { error!("{}", e); },
        }
    }

    info!("Disconnecting..");
}
```
