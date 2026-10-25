# frigate-detect

A local demo of [Frigate NVR](https://frigate.video), the open-source network video recorder with local AI object detection. Instead of a real camera, a looping video file is published as a fake RTSP camera, so you can explore live view, detection, zones and recorded events on any machine with Docker — no camera, no cloud.

## What it shows

- **Live view** of the `demo` camera in the Frigate UI.
- **Person and car detection** with bounding boxes (enable *Bounding boxes* in the camera settings).
- **Zones**: objects entering the `entrance` zone become *alerts*; everything else is kept as a *detection*.
- **Events with clips**: the *Review* page lists alerts/detections with thumbnails and plays the recorded clip; *Explore* lets you search snapshots by label.

## Architecture

```
video file (./video/*.mp4)
  -> ffmpeg loop (publisher: -re -stream_loop -1, copies H.264, otherwise re-encodes with libx264)
  -> MediaMTX (rtsp://mediamtx:8554/demo, internal only) -> Frigate 0.18.2 (CPU detector)
```

## Requirements

- Docker with Docker Compose

## Quick start

1. Put your own `.mp4` into `./video/`.
2. Set `VIDEO_FILE` in `docker-compose.yml` (publisher service) to its file name. Leave it empty to use the first `.mp4` in `./video` alphabetically.
3. Start the stack:
   ```sh
   docker compose up -d
   ```
4. Open http://localhost:5000. If port 5000 is taken by the macOS AirPlay Receiver, use http://127.0.0.1:5000.

Useful commands:

```sh
docker compose ps                          # status
docker compose logs -f frigate publisher   # logs
docker compose down                        # stop (recordings in ./media are kept)
```

After swapping the video, restart the publisher and Frigate (Frigate re-probes the detect resolution on start):

```sh
docker compose up -d publisher && docker compose restart frigate
```

Zone coordinates are relative (0–1) in `config/config.yml` and can be edited visually in the UI under *Settings → Masks / Zones*. Recordings and snapshots are retained for 1 day.

## About the video

This repository does not include a video. Use your own footage or a royalty-free clip (check its license). Footage with clearly visible people or cars from a fixed camera (street, parking lot, entrance) works best. The demo is meant for test footage, not for surveillance of real people.

## Slowed / timelapse footage

Timelapse clips move too fast for reliable tracking. This setup slowed the original clip down 4x with ffmpeg from the Frigate image:

```sh
docker run --rm -v "$PWD/video:/video" --entrypoint ffmpeg ghcr.io/blakeblackshear/frigate:0.18.2 -i /video/original/IN.mp4 -an -dn -vf setpts=4.0*PTS -r 15 -c:v libx264 -preset veryfast -crf 23 -pix_fmt yuv420p /video/OUT.mp4
```

## Production note

This demo uses the CPU detector. For real deployments, a Hailo-8/8L accelerator is recommended over the Coral TPU, because Google has archived the Gasket driver that the Coral PCIe/M.2 modules depend on.

## Security note

Frigate's port is bound to localhost only (`127.0.0.1` and `::1`), and MediaMTX is not published at all. Frigate has no authentication in this setup — do not expose it to the internet.

## License

MIT — see [LICENSE](LICENSE).
