# Sample Data retrieval from FactSet API



## Prerequisites

- Create API Key

You need to create an API key for you in the site: https://developer.factset.com/api-authentication

Use your email as "Name" so it is easily identifiable.

Once finish create a `.env` file in the parent directory as:

```bash
USERNAME="CPH_YOUR_PERSONAL_USER_NAME"
API-KEY="A1bc_YOUR_PERSONAL_API_PASSWORD"
```

Please note that this are secrets, do not upload them to any public repository or share them with colleages.

- Install dependencies

It is strongly recommended to use a Python package manager. Below, I will use [miniconda](https://www.anaconda.com/docs/getting-started/miniconda/main).


```bash
(base) ➜ conda env create -f environment.yml
(base) ➜ conda activate factset
```

If you use other Python package manager (e.g `uv`), please install the python packages required in `environment.yml` - otherwise you won't be able to get any data.
