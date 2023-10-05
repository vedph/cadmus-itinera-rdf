# Setup

- [Setup](#setup)
  - [Requirements](#requirements)
  - [Windows](#windows)
  - [Ubuntu](#ubuntu)
  - [Test](#test)

Here you will find the steps to setup graph for an Itinera database.

## Requirements

You will need:

(1) Itinera resources:

- [Itinera Docker compose script](https://raw.githubusercontent.com/vedph/cadmus-itinera-app/master/docker-compose.yml)
- [Itinera seed profile](https://raw.githubusercontent.com/vedph/cadmus-itinera-api/master/CadmusItineraApi/wwwroot/seed-profile.json)

(2) Itinera presets:

- [nodes.json](code/nodes.json)
- [triples.json](code/triples.json)
- [work-mappings.json](code/work-mappings.json)

(3) Cadmus CLI tool and its Itinera plugin:

- [the Cadmus CLI tool](https://github.com/vedph/cadmus_tool/releases): just download the latest release for your OS.
- [Cadmus CLI Itinera plugin](http://www.fusisoft.it/xfer/cadmus/cli/plugins/Cadmus.Itinera.Services.zip). This is a temporary URL. If you can't find it, or you need an up to date version, just build the [Itinera core project](https://github.com/vedph/cadmus-itinera) and copy `Cadmus.Itinera.Services` files under the CLI `plugins` folder.

>All the folders used in this example are arbitrarily chosen and named. You can place and name files as you prefer.

## Windows

🎯 Purpose: setup a graph-enabled Itinera stack with preset data in Windows.

(1) create a `cadmus-itinera` folder in your desktop, and download in it:

- all the Itinera presets listed [above](#requirements);
- the [Itinera seed profile](https://raw.githubusercontent.com/vedph/cadmus-itinera-api/master/CadmusItineraApi/wwwroot/seed-profile.json);
- the [Docker compose script](https://raw.githubusercontent.com/vedph/cadmus-itinera-app/master/docker-compose.yml), renaming it to `docker-compose.yml`.

(2) download the [Cadmus CLI tool](https://github.com/vedph/cadmus_tool/releases) and unzip it in `C:/exe/cadmus-tool` (preserving its directory structure).

(3) download the [Cadmus CLI Itinera plugin](http://www.fusisoft.it/xfer/cadmus/cli/plugins/Cadmus.Itinera.Services.zip) and unzip it into the `Cadmus.Itinera.Services` folder inside the `plugins` folder of the CLI tool.

(4) launch Cadmus Itinera with `docker compose up` from your `cadmus-itinera` folder, and wait until the running log has finished.

(5) from the Cadmus CLI folder, run these CLI commands to expand the mappings, create the index database and fill it with some presets:

```ps1
cd C:\exe\cadmus-tool
./cadmus-tool graph-deref $HOME/Desktop/cadmus-itinera/ms-mappings.json $HOME/Desktop/cadmus-itinera/ms-mappings-d.json
./cadmus-tool graph-deref $HOME/Desktop/cadmus-itinera/person-mappings.json $HOME/Desktop/cadmus-itinera/person-mappings-d.json
./cadmus-tool graph-deref $HOME/Desktop/cadmus-itinera/work-mappings.json $HOME/Desktop/cadmus-itinera/work-mappings-d.json
./cadmus-tool graph-import $HOME/Desktop/cadmus-itinera/nodes.json cadmus-itinera-graph -g repository-provider.itinera
./cadmus-tool graph-import $HOME/Desktop/cadmus-itinera/triples.json cadmus-itinera-graph -g repository-provider.itinera -m t
./cadmus-tool graph-import $HOME/Desktop/cadmus-itinera/ms-mappings-d.json cadmus-itinera-graph -g repository-provider.itinera -m m
./cadmus-tool graph-import $HOME/Desktop/cadmus-itinera/person-mappings-d.json cadmus-itinera-graph -g repository-provider.itinera -m m
./cadmus-tool graph-import $HOME/Desktop/cadmus-itinera/work-mappings-d.json cadmus-itinera-graph -g repository-provider.itinera -m m
```

(6) now you can open your browser at `localhost:4200`, and create a work item with a metadata part and an events part. New nodes and triples should appear (see [below](#test)).

🛠️ As an alternative, you can use these **Powershell commands** (one at a time, or create a batch):

```ps1
# download resources in desktop\cadmus-itinera
md $HOME/Desktop/cadmus-itinera
cd $HOME/Desktop/cadmus-itinera
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/vedph/cadmus-itinera-rdf/master/code/nodes.json" -OutFile nodes.json
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/vedph/cadmus-itinera-rdf/master/code/triples.json" -OutFile triples.json
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/vedph/cadmus-itinera-rdf/master/code/ms-mappings.json" -OutFile ms-mappings.json
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/vedph/cadmus-itinera-rdf/master/code/person-mappings.json" -OutFile person-mappings.json
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/vedph/cadmus-itinera-rdf/master/code/work-mappings.json" -OutFile work-mappings.json
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/vedph/cadmus-itinera-api/master/CadmusItineraApi/wwwroot/seed-profile.json" -OutFile seed-profile.json
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/vedph/cadmus-itinera-app/master/docker-compose_graph.yml" -OutFile "docker-compose.yml"

# download CLI tool in C:\exe\cadmus-tool (CHANGE VERSION AS REQUIRED)
C:
md exe
cd exe
md cadmus-tool
cd cadmus-tool
Invoke-WebRequest -Uri "https://github.com/vedph/cadmus_tool/releases/download/v.8.0.0/App-v.8.0.0-win-x64.zip" -OutFile cadmus-tool.zip
Extract-Archive cadmus-tool.zip -DestinationPath C:\exe\cadmus-tool\
del cadmus-tool.zip

# download CLI Itinera plugin
cd C:\exe\cadmus-tool
Invoke-WebRequest -Uri "http://www.fusisoft.it/xfer/cadmus/cli/plugins/Cadmus.Itinera.Services.zip" -OutFile Cadmus.Itinera.Services.zip
Extract-Archive Cadmus.Itinera.Services -DestinationPath c:\exe\cadmus-tool\plugins\Cadmus.Itinera.Services\
del Cadmus.Itinera.Services.zip

# fire up Cadmus Itinera
cd $HOME\Desktop\cadmus-itinera
docker compose up

# once startup has finished, open another Powershell window and continue...

# import presets
cd C:\exe\cadmus-tool
./cadmus-tool graph-deref $HOME/Desktop/cadmus-itinera/ms-mappings.json $HOME/Desktop/cadmus-itinera/ms-mappings-d.json
./cadmus-tool graph-deref $HOME/Desktop/cadmus-itinera/person-mappings.json $HOME/Desktop/cadmus-itinera/person-mappings-d.json
./cadmus-tool graph-deref $HOME/Desktop/cadmus-itinera/work-mappings.json $HOME/Desktop/cadmus-itinera/work-mappings-d.json
./cadmus-tool graph-import $HOME/Desktop/cadmus-itinera/nodes.json cadmus-itinera-graph -g repository-provider.itinera
./cadmus-tool graph-import $HOME/Desktop/cadmus-itinera/triples.json cadmus-itinera-graph -g repository-provider.itinera -m t
./cadmus-tool graph-import $HOME/Desktop/cadmus-itinera/ms-mappings-d.json cadmus-itinera-graph -g repository-provider.itinera -m m
./cadmus-tool graph-import $HOME/Desktop/cadmus-itinera/person-mappings-d.json cadmus-itinera-graph -g repository-provider.itinera -m m
./cadmus-tool graph-import $HOME/Desktop/cadmus-itinera/work-mappings-d.json cadmus-itinera-graph -g repository-provider.itinera -m m
```

>⚠️ Dereferencing mappings is required when your source file is not a JSON array with mappings, but rather a JSON object with named and document mappings sections. Also, when using `graph-import` command, you can add `-d` for dry-mode.

## Ubuntu

🎯 Purpose: setup a graph-enabled Itinera stack with preset data in Ubuntu.

(1) create a `cadmus-itinera` folder in your Documents folder, and download in it:

- all the Itinera presets listed [above](#requirements);
- the [Itinera seed profile](https://github.com/vedph/cadmus-itinera-api/blob/master/CadmusItineraApi/wwwroot/seed-profile.json);
- the [Docker compose script](https://github.com/vedph/cadmus-itinera-app/blob/master/docker-compose_graph.yml), renaming it to `docker-compose.yml`.

```bash
cd ~/Documents
mkdir cadmus-itinera
cd cadmus-itinera
wget https://raw.githubusercontent.com/vedph/cadmus-itinera-rdf/master/code/nodes.json
wget https://raw.githubusercontent.com/vedph/cadmus-itinera-rdf/master/code/triples.json
wget https://raw.githubusercontent.com/vedph/cadmus-itinera-rdf/master/code/ms-mappings.json
wget https://raw.githubusercontent.com/vedph/cadmus-itinera-rdf/master/code/person-mappings.json
wget https://raw.githubusercontent.com/vedph/cadmus-itinera-rdf/master/code/work-mappings.json
wget https://raw.githubusercontent.com/vedph/cadmus-itinera-api/master/code/CadmusItineraApi/wwwroot/seed-profile.json
wget https://raw.githubusercontent.com/vedph/cadmus-itinera-app/master/code/docker-compose_graph.yml
mv docker-compose_graph.yml docker-compose.yml
```

(2) in your `Documents` folder, create a `cadmus-tool` folder and enter it, download the Cadmus CLI tool for Ubuntu, and extract it. Also, download [http://www.fusisoft.it/xfer/Cadmus.Itinera.Services.zip](http://www.fusisoft.it/xfer/Cadmus.Itinera.Services.zip), i.e. the CLI plugin for Itinera, and unzip it under the `plugins` folder:

```bash
cd ~/Documents
mkdir cadmus-tool
wget https://github.com/vedph/cadmus_tool/releases/download/v.8.0.9/App-v.8.0.9-linux-x64.tar.gz
tar -xvzf --strip-components=1 App-v.8.0.9-linux-x64.tar.gz
cd plugins
wget http://www.fusisoft.it/xfer/Cadmus.Itinera.Services.zip
unzip Cadmus.Itinera.Services.zip
rm Cadmus.Itinera.Services.zip
```

(3) launch Cadmus Itinera with `docker compose up` from your `cadmus-itinera` folder, and wait until the running log has finished.

```bash
cd ~/Desktop/cadmus-itinera
sudo docker compose up
```

(4) from the Cadmus CLI folder, run these CLI commands to expand the mappings, create the index database and fill it with some presets:

```bash
./cadmus-tool graph-deref ~/Documents/cadmus-itinera/ms-mappings.json ~/Documents/cadmus-itinera/ms-mappings-d.json
./cadmus-tool graph-deref ~/Documents/cadmus-itinera/person-mappings.json ~/Documents/cadmus-itinera/person-mappings-d.json
./cadmus-tool graph-deref ~/Documents/cadmus-itinera/work-mappings.json ~/Documents/cadmus-itinera/work-mappings-d.json
./cadmus-tool graph-import ~/Documents/cadmus-itinera/nodes.json cadmus-itinera-graph -g repository-provider.itinera
./cadmus-tool graph-import ~/Documents/cadmus-itinera/triples.json cadmus-itinera-graph -g repository-provider.itinera -m t
./cadmus-tool graph-import ~/Documents/cadmus-itinera/ms-mappings-d.json cadmus-itinera-graph -g repository-provider.itinera -m m
./cadmus-tool graph-import ~/Documents/cadmus-itinera/person-mappings-d.json cadmus-itinera-graph -g repository-provider.itinera -m m
./cadmus-tool graph-import ~/Documents/cadmus-itinera/work-mappings-d.json cadmus-itinera-graph -g repository-provider.itinera -m m
```

(5) now you can open your browser at `localhost:4200`, and create a work item with a metadata part and an events part. New nodes and triples should appear (see [below](#test)).

## Test

1. create a new work item.
2. add a metadata part to the item and add metadatum `eid` with value `alpha`.
3. add en events part to the item and add en event of type send, with some mock data.
4. head to the graph UI and look at nodes and triples that were created.
