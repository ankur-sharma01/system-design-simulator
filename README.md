# System Design Simulator

An interactive, in-browser sandbox for learning and practising system design. Drag real-world components onto a blank canvas, wire them together, push traffic through the design, and watch where it bends, breaks and cascades. Built for interview prep and for building intuition about scale.

**Live demo:** `https://ankur-sharma01.github.io/system-design-simulator/`

It is a single `index.html` file with no build step, no dependencies and no backend.

---

## Features

- **Blank canvas.** Start from nothing and design anything, or open a ready-made template.
- **About 100 components** in a searchable palette: DNS, CDNs, load balancers, API gateways, compute, workers, caches, message queues, SQL and NoSQL databases, search, object storage, multi-region pieces, and observability and ops boxes.
- **Custom box.** Add your own component with a name, emoji, capacity, latency, cost and behaviour (pass-through service, terminal datastore, or showcase-only).
- **Live simulation.** Traffic flows through your graph every 500 ms. Nodes turn green, amber or flashing red as they load up, and flow lines show where traffic goes.
- **Metrics.** P50 and P99 latency, success rate, estimated monthly cost, and trend lines.
- **"Why it failed" panel.** When the error rate passes 5%, it names the bottleneck and suggests a fix. Click the blue suggestion and the app applies the fix for you and lists exactly what it changed.
- **40 library entries** grouped by category (see below). Every template uses a specific tech stack, and the **Tech stack** panel explains why each piece was chosen.
- **Chaos testing.** Kill any node, kill a random node, flush a cache to trigger a stampede, or turn on traffic patterns (daily wave, ramp up, periodic spikes) and a 10-second 3x burst.
- **Multi-region.** Health-checked failover routing, and cross-region database replication (async with replication lag, or synchronous with added write latency).
- **Autoscaling** toggle per node.
- **Canvas controls.** Zoom in and out, fit to view, fullscreen, mouse-wheel zoom, and drag the background to pan.
- **Export and import** designs as text so you can save them or share them with others.
- **Dark, black UI.**

## Quick start

1. Open the site (or `index.html` locally).
2. Drag a **Web users** node onto the canvas, then add components from the palette on the left.
3. Connect nodes by dragging from the blue dot on the right edge of a node onto another node. Click a connection to remove it.
4. Use the **Traffic** and **Writes** sliders at the top to change load and the read/write mix.
5. Select a node to tune it in the inspector (instances, cache hit rate, shards, replicas, consumer rate, routing and more).
6. When something goes red, read the panel at the bottom and click the blue suggestion to fix it.

Or open **📚 Templates** and pick a design, then press **🧱 Tech stack** to see the reasoning behind each technology.

## Template library

| Category | Templates |
|---|---|
| Multi-region and reliability | Active-active two regions, Global database (Spanner-style), Multi-region checkout |
| Social and messaging | News feed, Chat, Photo sharing, Notification system |
| Storage and media | Video streaming, File storage, Collaborative docs, Multi-tier CDN with origin shield, Media site with CDN |
| Search, maps and data | Search engine, Ad click aggregation, Maps and routing |
| Real-time systems | Stock trading platform, Video conferencing |
| Building blocks | URL shortener, Rate limiter, Distributed key-value store, Metrics and monitoring, Web crawler, Typeahead, Leaderboard, Event-driven writes, Microservices with service mesh |
| Commerce and marketplace | Ticket booking, E-commerce, Payment system, Ride hailing |
| AI infrastructure | LLM chat service, Recommendation engine |
| Databases and scaling | Read-heavy SQL with replicas, Message broker |
| Challenges | Flash sale surge, Viral news feed, Analytics pipeline, Budget startup, Cold cache stampede, Global media launch |

Each challenge has pass/fail goals (success rate, P99 latency and cost limits) shown in the bottom bar.

## How the simulation works

This is a teaching model, not a benchmark. The numbers are illustrative and chosen to show realistic behaviour, not to predict real hardware.

- **Traffic propagation.** Requests flow through the graph in topological order, split into reads, writes and async writes. Load balancers and gateways split evenly across their outputs, or skip dead targets with health-checked failover.
- **Utilisation.** Each node has a capacity per instance. Utilisation is load divided by capacity. Below 70% is healthy, 70% to 99% is a warning, and 100% or more is overloaded: requests are dropped and latency climbs steeply.
- **Caches.** Misses fall through to the datastore. The effective hit rate drops as the write share grows, and a freshly flushed cache warms up over time.
- **Queues.** Writes are accepted into a buffer and drained at a configurable consumer rate. A full buffer drops messages.
- **App servers and workers.** Capacity follows Little's law: each instance has 1,000 threads, and each request holds a thread for its own processing time plus the time spent waiting on downstream services. A slow database therefore starves the app tier and the failure cascades upstream.
- **SQL databases.** Default limits are about 5,000 reads/s per node and 2,500 writes/s per primary, with a connection pool. Read replicas add read capacity; shards multiply read and write capacity and cost.
- **NoSQL and other stores.** Per-instance throughput, with a strong-consistency option that halves throughput.
- **Latency.** Expected latency is summed along the request path using each node's base latency and load. P99 is an approximation derived from P50 and the most loaded node.
- **Cost.** Instances multiplied by an hourly rate multiplied by 730 hours per month.
- **Auto-fix.** Revives dead nodes, then repeatedly scales the most overloaded node (instances, replicas, shards, consumer rate, cache hit rate or rate limit) until the error rate is 1% or lower or it hits a limit. It sizes for the current load, so a later spike can still break the design.

## Running locally

No install is needed. Either open `index.html` in a browser, or serve the folder:

```bash
python3 -m http.server 8000
# then visit http://localhost:8000
```

## Deploying to GitHub Pages

1. Put `index.html` (and this README) in a GitHub repository.
2. Go to **Settings → Pages**.
3. Under "Build and deployment", choose **Deploy from a branch**, then select `main` and `/ (root)`, and save.
4. Your site will be live at `https://<your-username>.github.io/<repo-name>/` after a minute or two.

To update it later, upload or push a new `index.html` with the same name, then hard-refresh the page (Ctrl+Shift+R or Cmd+Shift+R) to bypass the browser cache.

## Project structure

```
.
├── index.html   # the entire app: HTML, CSS and JavaScript
└── README.md
```

Under the hood it uses plain JavaScript and SVG. There is no framework, no bundler and no external network requests.

## Tips for interviews

- Start from the **Blank canvas** and talk through each component choice as you add it.
- Open a template, press **Tech stack**, and rehearse the "why this technology" answers.
- Break your own design on purpose: use Chaos, flush the cache, apply a spike pattern, then explain what you would change.
- Use **Export** to save your own designs and **Import** to load someone else's.

## Limitations

- Components of the same kind share one behaviour model (for example Redis and Memcached behave alike); products differ in their capacity, latency and cost presets.
- Multi-region support covers failover routing and replication lag. It does not model network partitions or per-region traffic weighting.
- Designs are not saved automatically. Use Export before you close the page.
- Tech-stack explanations reflect common industry practice. They are a study aid, so verify details against current documentation before relying on them.

## License

Add a license of your choice (for example MIT) as a `LICENSE` file in the repository.
