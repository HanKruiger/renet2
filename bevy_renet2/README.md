# Bevy Renet
[![Latest version](https://img.shields.io/crates/v/bevy_renet2.svg)](https://crates.io/crates/bevy_renet2)
[![Documentation](https://docs.rs/bevy_renet2/badge.svg)](https://docs.rs/bevy_renet2)
![MIT](https://img.shields.io/badge/license-MIT-blue.svg)
![Apache](https://img.shields.io/badge/license-Apache-blue.svg)

A Bevy Plugin for the [renet2](https://github.com/UkoeHB/renet2) crate, forked from [renet](https://github.com/lucaspoffo/renet).
A network crate for Server/Client with cryptographically secure authentication and encrypted packets.
Designed for fast-paced competitive multiplayer games.

## Usage
Bevy renet is a small layer over the `renet2` crate, it adds systems to call the update function from the client/server. `RenetClient`, `RenetServer`, `NetcodeClientTransport` and `NetcodeServerTransport` need to be added as a resource, so the setup is similar to `renet2` itself:

#### Server
```rust
let mut app = App::new();
app.add_plugin(RenetServerPlugin);

let server = RenetServer::new(ConnectionConfig::default());
app.insert_resource(server);

// Transport layer setup
app.add_plugin(NetcodeServerPlugin);
let server_addr = "127.0.0.1:5000".parse().unwrap();
let socket = UdpSocket::bind(server_addr).unwrap();
let server_config = ServerSetupConfig {
    current_time: SystemTime::now().duration_since(SystemTime::UNIX_EPOCH).unwrap(),
    max_clients: 64,
    protocol_id: 0,
    socket_addresses: vec![vec![server_addr]],
    authentication: ServerAuthentication::Unsecure
};
let transport = NetcodeServerTransport::new(server_config, NativeSocket::new(socket).unwrap()).unwrap();
app.insert_resource(transport);

app.add_system(send_message_system);
app.add_system(receive_message_system);
app.add_system(handle_events_system);

// Systems

fn send_message_system(mut server: ResMut<RenetServer>) {
    let channel_id = 0;
    // Send a text message for all clients
    // The enum DefaultChannel describe the channels used by the default configuration
    server.broadcast_message(DefaultChannel::ReliableOrdered, "server message");
}

fn receive_message_system(mut server: ResMut<RenetServer>) {
    // Receive message from all clients
    for client_id in server.clients_id() {
        while let Some(message) = server.receive_message(client_id, DefaultChannel::ReliableOrdered) {
            // Handle received message
        }
    }
}

fn handle_events_system(mut server_events: EventReader<ServerEvent>) {
    for event in server_events.read() {
        match event {
            ServerEvent::ClientConnected { client_id } => {
                println!("Client {client_id} connected");
            }
            ServerEvent::ClientDisconnected { client_id, reason } => {
                println!("Client {client_id} disconnected: {reason}");
            }
        }
    }
}
```

#### Client
```rust
let mut app = App::new();
app.add_plugin(RenetClientPlugin);

let client = RenetClient::new(ConnectionConfig::default());
app.insert_resource(client);

// Setup the transport layer
app.add_plugin(NetcodeClientPlugin);

let authentication = ClientAuthentication::Unsecure {
    server_addr: SERVER_ADDR,
    client_id: 0,
    user_data: None,
    protocol_id: 0,
    socket_id: 0,
};
let socket = UdpSocket::bind("127.0.0.1:0").unwrap();
let current_time = SystemTime::now().duration_since(SystemTime::UNIX_EPOCH).unwrap();
let mut transport = NetcodeClientTransport::new(current_time, authentication, NativeSocket::new(socket).unwrap()).unwrap();

app.insert_resource(transport);

app.add_system(send_message_system);
app.add_system(receive_message_system);

// Systems

fn send_message_system(mut client: ResMut<RenetClient>) {
    // Send a text message to the server
    client.send_message(DefaultChannel::ReliableOrdered, "server message");
}

fn receive_message_system(mut client: ResMut<RenetClient>) {
    while let Some(message) = client.receive_message(DefaultChannel::ReliableOrdered) {
        // Handle received message
    }
}
```

## Example

You can run the `simple` example with:

* Server: `cargo run --features="serde native_transport" --example simple -- server`
* Client: `cargo run --features="serde native_transport" --example simple -- client`

If you want a more complex example you can checkout the [demo_bevy](https://github.com/UkoeHB/renet2/tree/master/demo_bevy) sample:

[Bevy Demo.webm](https://user-images.githubusercontent.com/35241085/180664609-f8c969e0-d313-45c0-9c04-8a116896d0bd.webm)

## Bevy Compatibility

|bevy|bevy_renet2  |
|----|-------------|
|0.16|0.9.0 -      |
|0.15|0.0.7 - 0.8.1|
