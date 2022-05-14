# WUDO Swarm Protocol

## General info

- `long` = `32 bit`
- maximum message size seems to be `0x1F4000`
- the protocol follows a simple `length || value` encoding (`||` = concatenation)
- it's a torrent-like protocol
- based on what I've seen, it really looks like Microsoft wants to expand the protocol in the future

## Handshake

The handshake packet is the only one that follows a different structure.

It starts with a protocol "magic":

```
htonl(len("Swarm protocol")) || "Swarm protocol"
```

You can send anything instead of "Swarm protocol", but some Windows versions seem to have stricter checks.

Constraints on the string you send (based on my Windows 10 Home):

- `len(magic) > 0`
- `len(magic) != 0x16`
- `len(magic) <= 0x32`

After the magic, comes the protocol version: `00 00 00 01 00 00 00 00`.

This looks like version `1.0.0.0` encoded in a funny way.

Then comes the swarm hash (32 bytes).

Finally, the peer id (20 bytes).

You send this to a Windows machine and it will reply to you (**if** the target has the file identified by the swarm hash).

## Other messages

These messages all follow the structure `htonl(length(payload)) || payload`.

### Keep Alive

Zero-length packet.
So you just send the length without a payload:

`00 00 00 00`

### Choke

Raw message: `00 00 00 01 00`:

- length 1
- message type: 0

### Unchoke

Raw message: `00 00 00 01 01`

- length 1
- message type: 1

### Interested

Raw message: `00 00 00 01 02`

### Not interested

Raw message: `00 00 00 01 03`

### Have

Signals that a peer has a block of the file.

I will skip the initial 4 bytes of length from now on.

- message type: `04`
- offset: `htonl(offset of block in file)`

### Bit field

Signals which blocks a peer has (1 bit = 1 block).

- message type: `05`
- bitfield: `ceil(bitfield // 8)` bytes

### Cancel block

`B(idx, begin, len)`:

- message type: `08`
- index of block: `htonl(idx)`
- offset of block: `htonl(begin)`
- length of block: `htonl(len)`

### Request block

Ask a peer to send a block.

- message type: `06`
- index of block: `htonl(idx)`
- offset of block: `htonl(begin)`
- length of block: `htonl(len)`

### Start block

Send block to peer.

- message type: `07`
- index of block: `htonl(idx)`
- offset of block: `htonl(begin)`
- block: `len` bytes from the file (len is calculated using `payload size - 9`)

### Wtf message

Just consumes data?
Maybe it's for future purposes.

- message type: `14`
- data: `payload - 1` bytes that will be discarded