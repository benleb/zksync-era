# For using private GitHub dependencies, CI downdloads all crates outside of the contatiner
# Not expected to work locally

# syntax=docker/dockerfile:experimental
FROM debian:bookworm-slim as builder

ARG DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y curl clang openssl libssl-dev gcc g++ \
    pkg-config build-essential libclang-dev && \
    rm -rf /var/lib/apt/lists/*

ENV RUSTUP_HOME=/usr/local/rustup \
    CARGO_HOME=/usr/local/cargo \
    PATH=/usr/local/cargo/bin:$PATH

RUN curl https://sh.rustup.rs -sSf | bash -s -- -y && \
    rustup install nightly-2023-07-21 && \
    rustup default nightly-2023-07-21

WORKDIR /usr/src/zksync
COPY . .

RUN CARGO_HOME=./cargo cargo build --release

FROM debian:bookworm-slim
RUN apt-get update && apt-get install -y curl libpq5 ca-certificates && rm -rf /var/lib/apt/lists/*

# copy finalization hints required for witness vector generation
COPY prover/vk_setup_data_generator_server_fri/data/ /prover/vk_setup_data_generator_server_fri/data/

COPY --from=builder /usr/src/zksync/target/release/zksync_witness_vector_generator /usr/bin/
ENTRYPOINT ["zksync_witness_vector_generator"]
