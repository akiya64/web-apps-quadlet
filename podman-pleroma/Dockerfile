# https://github.com/angristan/docker-pleroma
# 上記をベースに、Google の生成AIにマルチステージビルドに書き直させた.
#
# ==========================================
# ステージ1: ビルド用環境 (builder)
# ==========================================

FROM elixir:1.20-slim AS builder

ARG PLEROMA_VER=develop

ENV MIX_ENV=prod
ENV VIX_COMPILATION_MODE=PLATFORM_PROVIDED_LIBVIPS

# コンパイルに必要な開発ツールをインストール
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    git \
    gcc \
    g++ \
    make \
    cmake \
    libmagic-dev \
    libncurses5-dev \
    ca-certificates \
    erlang-dev \
    libvips-dev \
    pkg-config \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /pleroma_build

# ソースコードをクローンしてコンパイル
RUN git clone -b develop https://git.pleroma.social/pleroma/pleroma.git . \
    && git checkout ${PLEROMA_VER}

RUN echo "import Mix.Config" > config/prod.secret.exs \
    && mix local.hex --force \
    && mix local.rebar --force \
    && mix deps.get --only prod \
    && mkdir /pleroma_release \
    && mix release --path /pleroma_release


# ==========================================
# ステージ2: 実行用環境 (runner)
# ==========================================

FROM debian:trixie-slim AS runner

ARG UID=1000
ARG GID=1000
ARG DATA=/var/lib/pleroma

ENV MIX_ENV=prod
ENV LANG=C.UTF-8
ENV LC_ALL=C.UTF-8

# 実行時に最低限必要なランタイム・ライブラリのみをインストール
# (gccやmake、elixir本体などはここには含まれません)
RUN apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates \
    libmagic1 \
    libncurses6 \
    libvips42 \
    libsctp1 \
    && rm -rf /var/lib/apt/lists/*

# ユーザーとディレクトリの設定
RUN groupadd -g ${GID} pleroma \
    && useradd -d /pleroma -s /bin/false -g pleroma -u ${UID} -m pleroma

RUN mkdir -p /etc/pleroma \
    && chown -R pleroma /etc/pleroma \
    && mkdir -p ${DATA}/uploads \
    && mkdir -p ${DATA}/static \
    && chown -R pleroma ${DATA}

USER pleroma
WORKDIR /pleroma

# ステージ1 (builder) で生成されたコンパイル済み成果物だけをコピー
COPY --from=builder --chown=pleroma /pleroma_release /pleroma
COPY --chown=pleroma --chmod=640 ./config.exs /etc/pleroma/config.exs

EXPOSE 4000

CMD ["/pleroma/bin/pleroma", "start"]

