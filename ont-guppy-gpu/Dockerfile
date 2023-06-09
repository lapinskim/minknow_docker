## -*- dockerfile-image-name: "ont-guppy-gpu" -*-
# syntax=docker/dockerfile:1
FROM nvidia/cuda:11.6.2-base-ubuntu20.04

ENV TZ=Europe/Warsaw

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates \
    curl \
    gnupg2 && \
    curl -fsSL https://cdn.oxfordnanoportal.com/apt/ont-repo.pub | apt-key add - && \
    echo "deb http://cdn.oxfordnanoportal.com/apt focal-stable non-free" \
    >/etc/apt/sources.list.d/nanoporetech.sources.list && \
    apt-get purge --autoremove -y curl \
    && rm -rf /var/lib/apt/lists/*

ARG GUPPY_VERSION=6.4.6-1~focal

RUN apt-get update && apt-get install -y --no-install-recommends \
    ont-guppy-for-minion="$GUPPY_VERSION" \
    && rm -rf /var/lib/apt/lists/*

ENV GUPPY_LOG_PATH=/var/log/minknow/guppy
ENV GUPPY_CONFIG=dna_r9.4.1_450bps_fast.cfg
ENV GUPPY_NUM_CALLERS=1
ENV GUPPY_CPU_THREADS_PER_CALLER=6
ENV GUPPY_PORT=/tmp/.guppy/5555
ENV GUPPY_IPC_THREADS=3
ENV GUPPY_DEVICE=cuda:all:100%
ENV GUPPY_GPU_RUNNERS_PER_DEVICE=4
ENV GUPPY_CHUNKS_PER_RUNNER=320
ENV GUPPY_CHUNK_SIZE=2000

ENTRYPOINT ["/usr/bin/bash", "-c",\
    "exec /opt/ont/guppy/bin/guppy_basecall_server\
    --log_path\
    $GUPPY_LOG_PATH\
    --config\
    $GUPPY_CONFIG\
    --num_callers\
    $GUPPY_NUM_CALLERS\
    --cpu_threads_per_caller\
    $GUPPY_CPU_THREADS_PER_CALLER\
    --port\
    $GUPPY_PORT\
    --ipc_threads\
    $GUPPY_IPC_THREADS\
    --device\
    $GUPPY_DEVICE\
    --gpu_runners_per_device\
    $GUPPY_GPU_RUNNERS_PER_DEVICE\
    --chunks_per_runner\
    $GUPPY_CHUNKS_PER_RUNNER\
    --chunk_size\
    $GUPPY_CHUNK_SIZE"\
]
