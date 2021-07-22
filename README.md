# Archenland

### A distributed Blender network renderfarm and web interface

Archenland is comprised of two major parts: the central server which manages distributed rendering, and any number of worker clients which individually render frames of the animation.

Archenland strives to be as dynamic as possible, but is targeted at smaller-size operations, and is not designed for use as a commercial renderfarm. It currently dynamically balances rendering load equally across all workers for a single job, and does not support running jobs in parallel, featuring a queue system instead.

## Initial Setup

To use Archenland, you will need the following installed:

- Blender (Required on worker processes only)
- Python
- `aiohttp` and `websockets` for Python
- ffmpeg (Required on central server only)

### **For Central Server Setup:**

In `settings.conf`:
- Set `ffmpeg_path` properly to ffmpeg executable; if using a system-wide installation of ffmpeg, setting it to simply `"ffmpeg"` will work.
- Set `port` (used as the websocket port to communicate data with the worker processes and exchange updates with web interface users)
- Set `web_port` (used as the HTTP server port, to serve `.blend` files to workers, recieve completed frames, and handle web interface requests.)

**Example Central Server `settings.conf`**:
```
ffmpeg_path = "ffmpeg"
port = 7411
web_port = 8080
```

### **For Worker Process Setup:**

In `settings.conf`:
- Set `blender_path` properly to the Blender executable; if using a system-wide installation of Blender, setting it to simply `"blender"` will work.
- Set `threads` to the number of threads you wish Blender to use for rendering on this worker process. Setting this value to `0` will let Blender itself decide (likely resulting in the use of all available threads.)
- Set `address` to the address of the central server. For example, if the worker is running on the same computer as the central server, the address will be `"127.0.0.1"`.
- Set `port` to the same value as it was set on the central server.
- Set `web_port` to the same value set on the central server.

**Example Worker Process `settings.conf`**:
```
blender_path = "blender"
threads = 2
address = "127.0.0.1"
port = 7411
web_port = 8080
```

## Usage

To set up renderfarm control, start a single central server process:
```
python server.py
```

Then, for each worker process, run:
```
python worker.py
```

You may then access the web interface at `<central server address>:<web_port>` in your browser, replacing `<central server address>` with the address to the central server, and `<web_port>` with the `web_port` you set in `settings.conf`.

Once at the web interface, you may upload a `.blend` file to begin a rendering job. Archenland will automatically balance rendering load across all worker clients, to deliver the finished render in the least time possible.

#

## Implementation Notes

For those curious as to how Archenland functions, or those interested in contributing (thank you!), a rough sketch of Archenland's inner workings is given below.