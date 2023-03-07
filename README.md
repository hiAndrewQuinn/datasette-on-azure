# Deploying Datasette using Azure Functions

This repository shows an example of Datasette deployed using Azure Functions.

https://azure-functions-datasette.azurewebsites.net/

It uses a slightly modified version of the ASGI proof of concept wrapper created by Anthony Shaw in [this GitHub issues thread](https://github.com/Azure/azure-functions-python-library/issues/75#issuecomment-808553496).

To deploy you'll need an Azure account, plus the `az` and `func` command-line tools. I installed these following the instructions in [this tutorial](https://docs.microsoft.com/en-us/azure/azure-functions/create-first-function-cli-python?tabs=azure-cli%2Cbash%2Cbrowser) - specifically [this](https://docs.microsoft.com/en-us/azure/azure-functions/functions-run-local?tabs=macos%2Ccsharp%2Cbash#v2) and [this](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli).

You'll need to create a resource group, a storage account and a function app.

I found that the resource group needed to be in `westeurope` for functions to work correctly.

Creating the resource group:

```bash
az group create \
    --name datasette-rg \
    --location westeurope
```

Creating the storage account:

```bash
az storage account create \
    --name datasettestorage \
    --location westeurope \
    --resource-group datasette-rg \
    --sku Standard_LRS
```

Creating the function app. The name here is the name that will be exposed in the default URL, so it needs to be globally unique - unlike the storage account and resource group which only have to be unique within your Azure account

```bash
az functionapp create \
    --resource-group datasette-rg \
    --consumption-plan-location westeurope \
    --runtime python \
    --runtime-version 3.10 \
    --functions-version 4 \
    --storage-account datasettestorage \
    --os-type linux \
    --name azure-functions-datasette
```

In Simon's original draft, there's an explicit `publish` step, but with Functions version 4 it looks like that might no longer be necessary? Well I'm up and running at https://azure-functions-datasette.azurewebsites.net/ in any case now.

```bash
curl https://azure-functions-datasette.azurewebsites.net/
```

Will return tomorrow to keep at it.

===

We'll need both `az` and `func` for this next part.

These commmands will fill `loca.settings.json` with the right
stuff to get started.

```bash
func azure functionapp fetch-app-settings azure-functions-datasette
func azure storage fetch-connection-string datasettestorage
```

We get a "referenced bundle does not meet required minimum version"
error, and this means we need to change something in `host.json`.

... Gah, now `func start` works but we're getting a bunch of errors.
Seems like we'll need a venv.

```bash
python -m venv venv
source venv/bin/activate
```

... close... I think I need to install the things in `requirements.txt`
as well. We can use pipenv for that.

```bash
# If you're on Ubuntu, remove the apt version, it's way too old
# and will probably throw a weird mutablemapping error
sudo apt remove pipenv
pip install pipenv

pipenv install
pipenv shell

func start
```

Hell yeah! No errors!!

Finally let's publish it.

```bash
func azure functionapp publish azure-functions-datasette
```
