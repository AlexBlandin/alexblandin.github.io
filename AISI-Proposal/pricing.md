
# Tentative pricing for drafting a budget (2024-10-22)

A good example of some more realistic cloud-native (spot) pricing is given in [RETRO Is Blazingly Fast](https://mitchgordon.me/ml/2022/07/01/retro-is-blazing.html) where the cost (~$1k) was largely dominated by re-embedding the entire dataset (note, they did not re-train):

> Tokenization takes around 1.9 min / 1M chunks on your standard CPU core. The Pile ends up being around 5.8B chunks (370B tokens), so that means you’re looking at ~180 hours of CPU time to tokenize, which you can easily parallelize down to only a few hours of wall time.
>
> With a CPU core on the cloud going for around $0.03 / hour, that means you’ll spend less than $10 on tokenization.
>
> BERT embedding is the most expensive step. On an RTX A5000, BERT embedding takes around 10 minutes per 1M chunks. That’s around 1k GPU hours to embed The Pile, which again is trivial to parallelize. This cost around $1k on CoreWeave.
>
> Note that BERT embeddings are around 3 KB each on disk. (768 float32s). 5.8B of them takes up about 16 TB on disk, so watch out for that. (Disk space is cheap.)
>
> One particular trick used by FAISS (the inverted file structure) requires taking a small percentage of the data (64M embeddings) and using them to train the index. On a V100 GPU, this only took around 4 hours, so the cost was negligible.
>
> Once the index is trained, we can add all the embeddings to the index, compressing them for lookup. This takes longer than you’d expect (around 192 CPU hours) but ultimately only represents a cost of <$30.
>
> The FAISS index is not totally cost free. The index itself ends up being big, requiring around 176 GB of RAM to query, which costs about $0.88 per hour on your average cloud provider.
>
> However, this allows you to drastically reduce your GPU usage. Say, for example, you need 5 GPUs running in parallel to do inference on a 175B parameter model, which costs around $6 an hour. By adding an extra $0.88 / hour in CPU RAM, you can reduce the number of GPUs you have to run to just 1, saving around $5 / hour in GPU costs.
>
> This also applies to models that are already using a single GPU. By shrinking your model with RETRO’s database, requests get served faster, meaning more GPU bang for your buck. Instead of serving 60 req / hour on a single GPU, you’re serving 600+, just for a little extra CPU RAM.
>
> FAISS has the ability to memory map an index, which allows you to read it directly from disk instead of allocating RAM for it. This is slightly slower, of course, but probably worth the trade. (Thanks rom1504.)

- Expected Hardware Requirements (Initial Pass)
  - 20+ TB storage (size of RETRO embeddings, bigger vector DB & indices is good price/performance)
    - "best" managed as a JBOD, but ~4 HDDs (or SSDs) are easy to fit in a workstation/server
      - JBOD: "just a bunch of disks"
      - we will want some measure of redundancy (ZFS/RAID and cold spares for resilver/rebuild)
      - backups will need to be handled, Backblaze could be a good "quick and dirty" approach
  - ideally 256 GB RAM, minimum 64-128 GB, ECC RDIMMs are a must
  - ideally 32-64 vCPUs, potentially, minimum 24 vCPUs (16 cores or 8+16 cores)
  - sufficient bandwidth
  - 1x 20 GB GPU should suffice, can spot rent 4-8+ GPU configs if needed (AI/GPU cloud)
- Example "one box" workstation pricing
  - Stone (without VAT) (and no GPU! so not even "one box")
    - £3264.68 Lenovo ThinkStation P620
      - 64 GB of 1 TB, Ryzen Threadripper PRO 5965WX (48 vCPUs, 280 W)
    - £1706.29 Dell Precision 7865
      - 32 GB of 1 TB, Ryzen Threadripper PRO 5945WX (24 vCPUs, 280 W)
    - £1359.92 Lenovo ThinkStation P3
      - 32 GB of 128 GB, Intel Core i9 i9-14900K (32 vCPUs, 8 + 16 cores, 250 W)
    - £4076.31 HPE ProLiant DL380 Gen11 (2U)
      - 128 GB of 8 TB, Xeon Silver 4514Y (32 vCPUs, 150 W)
  - Direct from OEM (excluding VAT, for US sites currently $1 : £1.2~1.3)
    - £4,155.88 Dell Precision 5860
      - 128 GB of 2 TB, Intel Xeon W5-2465X (32 vCPUs, 200 W)
      - NVIDIA 4000 Ada
    - £5,070.84 Dell Precision 7875
      - 128 GB of 2 TB, AMD Ryzen Threadripper PRO 7955WX (32 vCPUs, 350 W)
      - NVIDIA 4000 Ada
    - £5,728.63 Dell Precision 7960
      - 128 of 2 TB, Intel Xeon w5-3433 (32 vCPUs, 220 W)
      - NVIDIA 4000 Ada
    - £6,665.43 Dell Precision 7960 Rack (2U)
      - 128 GB of 2 TB, Intel Xeon Silver 4514Y (32 vCPUs, 150 W)
      - NVIDIA 4000 Ada
    - $3,969.00 Lenovo ThinkStation P3 (~£5200)
      - 128 GB, Intel Core i9-13900 vPro (32 vCPUs, 8 + 16 cores, 220 W)
      - NVIDIA 4000 Ada
    - Workstation w/o a GPU is practical, OEMs have x1.5-x2.5 markup on GPUs
- "Build Your Own Box" Pricing
  - Workstation Tower or Rackmount Server?
    - Will likely just SSH/remote in from laptop either way, so, server may be the best choice
      - far from desk, co-lo if needed, easier to integrate hot-swappable redundant parts (PSU), UPS, networking, and cooling (fan noise somewhere else)
      - downside is "server/enterprise" grade components can be pricier
  - £0.00 Software Stack (it's all FOSS)
    - XCP-ng + Xen Orchestra as bare-metal host for Xen, primarily running FreeBSD 14 (ZFS) + bhyve
      - Xen and bhyve have full GPU passthrough for Debian & Alpine VMs (if not via ports/Linuxulator)
        - bhyve may need `vnc` enabled to give the driver a VGA buffer on FreeBSD and Windows guests
        - NVIDIA closed-source drivers will probably be essential, c'est la vie
    - Data Science / Processing
      - DuckDB, Postgres + `pgvector`, Ibis, NumPy, Numba, SciPy, PySR, FAISS
    - ML
      - Keras 3.0 (JAX, PyTorch, TensorFlow)
      - [JAX](https://github.com/n2cholas/awesome-jax)
        - Equinox, Elegy, Flax (NNX) & `flaxmodels`, Haiku, Trax, Jraph, Objax, Levanter, `sklearn-jax-kernels`, `efficientnet-jax`
        - FedJAX, Neural Tangents, `ensemble_net` & `ood_focal_loss`, Lorax, AQT (Accurate Quantized Training)
        - `f_net` FNet, T5X, `MaxText`, GPT-J
        - chex, Haliax, `CommonLoopUtils`, `JAX-tqdm`
        - Optax, Diffrax, Optimistix, Lineax, `sympy2jax`, `scalable_shampoo` Shampoo, `generax`, `jax-flows`, `parallel-non-linear-gaussian-smoothers` (Kalman Filtering)
        - NumPyro, BlackJAX, Distrax, Bayex, Bayes-Newton, `JAXNS`, Oryx, MCX, `jxf`
        - RLax, PureJaxRL, coax, `Mctx`, Jumanji
      - interpretML (EBM, DiCE, GAM Changer), scikit-learn / sklearn
      - Kolmogorov-Arnold Networks (KAN)
        - `pykan`, `efficient-kan`, `FourierKAN`, `GraphKAN`, `kanrl`
    - Visualisation
      - HoloViz (handles Bokeh, Matplotlib, and Plotly, supports seaborn and graphviz)
  - ~£7000, ~£4800 in "compute", ~£1700 in storage, ~£500 for chassis & power
    - £1,238.99 CPU (assuming DDDR5, older chips with DDR4 will be much cheaper, also used available)
      - [SCAN AMD CPUs](https://www.scan.co.uk/shop/computer-hardware/cpu-amd-server/all)
        - £672.49 AMD EPYC 4584PX (32 vCPUs, 120 W, Zen 4c, AM5 dual-channel, maybe for a CPU farm?)
        - £659.99 AMD EPYC 8124P (32 vCPUs, 125W, Zen 4c, SP5)
        - £920.99 AMD EPYC 9124 (32 vCPUs, 200 W, Zen 4)
        - £1,238.99 AMD EPYC 9135 (32 vCPUs, 200 W, Zen 5, [AVX512](https://justine.lol/matmul/))
        - £2,699.99 AMD EPYC 9355P (64 vCPUs, 280 W, Zen 5)
    - ~£1000 Motherboard
      - [SCAN SP5](https://www.scan.co.uk/shop/computer-hardware/motherboards-amd/all#t23.f2=SP5)
        - £994.99 Gigabyte AMD MZ33-AR0
      - [SCAN AM5](https://www.scan.co.uk/shop/computer-hardware/motherboards-amd/all#t23.f2=AM5)
        - £169.99 ASUS PRIME B650-PLUS
        - £240.98 ASUS PRIME X870-P
    - £1,339.99 NVIDIA RTX 4000 Ada (20 GB, 130 W)
      - [SCAN Workstation GPUs](https://www.scan.co.uk/shop/pro-graphics/nvidia-workstation-gpus/all)
        - £1,339.99 NVIDIA RTX 4000 Ada (20 GB, 130 W, 6144 CUDA, 192 Tensor)
        - £2,339.99 NVIDIA RTX 4500 Ada (24 GB, 210 W, 7680 CUDA, 240 Tensor)
          - Matches NVIDIA L4
        - £7,199.99 NVIDIA RTX 6000 Ada (48 GB, 300 W, 18176 CUDA, 568 Tensor)
          - Matches NVIDIA L40
          - Not the NVIDIA L40S (350 W variant that doubles Tensor performance)
      - [SCAN Desktop GPUs](https://www.scan.co.uk/shop/computer-hardware/gpu-nvidia-gaming/all)
        - £969.98 MSI NVIDIA GeForce RTX 4080 SUPER (16 GB, 320 W, 10240 CUDA, 320 Tensor)
    - ~£1200 DDR5 ECC RDIMMs (256 GB)
      - [SCAN DDR5 Server Kits](https://www.scan.co.uk/shop/computer-hardware/memory-ram/ddr5-server-ram-memory-kits-5600)
        - £300 1x64 GB DDR5 ECC RDIMM (4800-5600 MT/s)
          - £600 for 128 GB, £1200 for 256 GB, £4800 for 1 TB
      - [SCAN DDR4 Server Kits](https://www.scan.co.uk/shop/computer-hardware/memory-ram/3200mhz-ddr4-server-ram-memory-kits)
        - £170 1x64 GB DDR4 ECC RDIMM (3200 MT/s)
          - £340 for 128 GB, £680 for 256 GB, £2720 for 1 TB
    - 20 TB + 10 TB redundancy, £800 in storage (HDDs) or £1500 in storage (SSDs)
      - £1,680 7x WD Blue SN5000 (4 TB) for 20 TB volume with 8 TB parity or parity + cold (example)
      - ~5 TB parity, ~5 TB cold spare (hot-swap and resilver/rebuild with)
      - see [Disk Prices](https://diskprices.com/?locale=uk) and [Mouser](https://www.mouser.co.uk)
        - £15-25/TB HDDs (£300-500 / 20 TB), would use mixed drives, [see Drive Stats](https://www.backblaze.com/cloud-storage/resources/hard-drive-test-data)
          - £235 Seagate Exos X18 Enterprise Class (16 TB, £14.69/TB)
          - £210 Seagate IronWolf Pro (14 TB, £15.00/TB)
          - £360 Toshiba MG Enterprise (22 TB, £16.36/TB)
          - £228 WD Red Plus NAS (10 TB, £22.81/TB)
          - £195 WD_BLACK D10 (8 TB, £24.38/TB)
        - £50-60/TB SSDs (£1000-2000 / 20 TB), better performance & reliability at higher cost
          - £190 Kingston NV2 (4 TB SSD, £47.50/TB)
          - £101 Samsung 990 EVO (2 TB, £50.60/TB)
          - £240 WD Blue SN5000 (4 TB, £60.0/TB)
          - £750 WD_Black SN850X (8 TB, £93.74/TB)
          - £3,232.91 Solidigm D5-P5336 (30.72 TB, £105.24/TB)
    - £100 PSU (for desktop/tower grade) or 2x £200 server PSU (though barebone servers come with)
      - [SCAN Desktop PSU](https://www.scan.co.uk/shop/computer-hardware/power-supplies/all)
        - £99.98 Seasonic Focus GX 850 (850W)
      - [Mouser Rackmount PSU](https://www.mouser.co.uk/c/power/power-supplies/rack-mount-power-supplies/)
        - £208.02 Delta Electronics CRPS (1300 W, Platinum 90, 1U)
    - £100 2U or 4U or Tower Chassis
      - [SCAN Barebone Servers](https://www.scan.co.uk/shop/computer-hardware/servers/all)
        - £2,538.49 Gigabyte R272-Z32 AMD EPYC 7002 Server (2U Rackmount, 24x NVMe Bays, 2x 1200W)
        - £2,742.98 ASUS RS520A-E12-RS24U AMD EPYC 9004 Server (2U Rackmount, 24x NVMe Bays, 2x 1600W)
    - £0.00 JBOD chassis (to store bulk HDDs or SSDs, if needed)
      - rack mount (may just fit right alongside/beneath server)
      - even just a 2U or 4U would be plenty for now, let alone a full shelf of disk "blades"
      - but 8x 4 TB M.2 easily fits in a regular server / workstation
        - see barebones with 24x 2.5" bays for NVMe, we can fit more than enough storage to begin with
  - ~£3-4k UPS, ideally runs 24 hours at-load (500-800 W) (power cut at night, can respond next day)
    - most UPS units are "small", only 1-2 hour @ 600 W, but enough for automatic shutdown
      - £2,850.00 APC Easy UPS On-Line SRV6KI Tower (1h 17 min @ 600 W, including VAT)
      - £4,050.00 APC Easy UPS On-Line SRV6KRI 4U Rack (1h 17 min @ 600 W, including VAT)
      - £4,140.00 APC Easy UPS On-Line SRV6KRILRK 4U Rack (1h 54 min @ 600 W, including VAT)
        - Supports 4 external SRV240RLBP-9A battery packs
    - so, (automatic) shutdown, logging, "checkpoint", use "autosave" snapshots for a delta-based "interrupt" checkpoint/snapshot, so long as we can reliably flush to disk, still relying on good snapshot and backup management
    - if we're at a co-lo of some description, this is likely something handled where they will have some X hours of backup generator fuel or batteries at the ready, though they might not be able to provide the hours of runtime, though they have some absolutely overbuilt power delivery anyway, so it takes a lot more for them to experience trouble at all
- AI/GPU Cloud Pricing
  - [CoreWeave](https://www.coreweave.com/gpu-cloud-pricing)
    - ...
  - [RunPod.io](https://www.runpod.io/pricing)
    - ...
- VPS Cloud Pricing
  - Hetzner (dedicated and volumes are stopped at account limits, can go higher)
    - €7.56/month Intel Xeon Gold (8 GB, 4 vCPUs, shared)
    - €65.28/month AMD EPYC 7002 (32 GB, 16 vCPUs, shared)
    - €57.59/month AMD EPYC 7003 or 9004 (32 GB, 8 vCPUs, dedicated)
    - €230.39/month AMD EPYC 7003 or 9004 (128 GB, 32 vCPUs, dedicated)
    - €52.80/month Volume (1 TB, €0.0528/GB/month, including VAT)
- Cloud Pricing
  - Azure
    - £836.89/month D32ds v4 (128 GB, 32 vCPUs)
    - £767.70/month NC16as T4 v3 (110 GB, 16 AMD EPYC 7v12 vCPUs, NVIDIA T4 (16 GB))
      - £488.88/month with 1yr reserved, £5,866.55 charged upfront
    - £430.4/month S70 HDD (16 TB)
    - £215.2/month S60 HDD (8 TB)
  - Google
    - £713.33/month g2-standard-16 (64 GB, 16 vCPUs, NVIDIA L4 (24 GB))
      - £449.40/month with 1yr reserved
    - £853.91/month g2-standard-16 custom (128 GB, 16 vCPUs, NVIDIA L4 (24 GB))
      - £533.64/month with 1yr reserved
    - £1,078.43/month g2-standard-32 (128 GB, 32 vCPUs, NVIDIA L4 (24 GB))
      - £679.41/month with 1yr reserved
    - £281.91/month Cloud Storage (16 TB)
