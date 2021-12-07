# blockscout
# Dependencies
wget https://packages.erlang-solutions.com/erlang-solutions_2.0_all.deb
sudo dpkg -i erlang-solutions_2.0_all.deb
sudo apt-get update
sudo apt-get install -y esl-erlang=1:23*

wget https://github.com/elixir-lang/elixir/releases/download/v1.11.0/Precompiled.zip
sudo unzip Precompiled.zip -d /opt/elixir

sudo apt-get install -y automake libtool libgmp3-dev inotify-tools postgresql postgresql-contrib npm

sudo apt update
curl -sL https://deb.nodesource.com/setup_14.x | sudo bash -
sudo apt -y install nodejs

curl https://sh.rustup.rs -sSf | sh -s -- -y
export RUSTUP_HOME=/opt/rust
export CARGO_HOME=/opt/rust
ln -s /opt/rust/bin/* /usr/local/bin/
ln -s /opt/elixir/bin/* /usr/local/bin/

# Build
git clone https://github.com/poanetwork/blockscout && cd blockscout

mix deps.get
mix phx.gen.secret

echo "
# blockscout/start.sh

export HOME=/home/{ YOUR_HOME } # DON'T FORGET CHANGE THIS
export ETHEREUM_JSONRPC_HTTP_URL=http://geth:8545  # DON'T FORGET CHANGE THIS
export ETHEREUM_JSONRPC_WS_URL=ws://geth:8546  # DON'T FORGET CHANGE THIS
export ETHEREUM_JSONRPC_TRACE_URL=http://geth:8545  # DON'T FORGET CHANGE THIS
export ETHEREUM_JSONRPC_VARIANT=geth
export COIN=ETH
export DATABASE_URL=postgresql://{ USER_NAME }:{ PASSWORD }@{ DB_HOST }:5432/{ DB_NAME } # DON'T FORGET CHANGE THIS
" > start.sh

mix do deps.get, local.rebar --force, deps.compile, compile
mix do ecto.create, ecto.migrate

cd apps/block_scout_web/assets && npm install && node_modules/webpack/bin/webpack.js --mode production && cd -
cd apps/explorer && npm install && cd -

mix phx.digest

cd apps/block_scout_web && mix phx.gen.cert blockscout blockscout.local && cd -
