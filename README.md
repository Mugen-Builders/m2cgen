<div align="center">
    <h1>m2cgen dApp</h1>
    <i>An Machine Learning example using Cartesi Rollups</i>
</div>
<div align="center">
This example shows a simple way of leveraging some of the most widely used Machine Learning libraries available in Python.
</div>

<div align="center">
  
  <a href="">[![Static Badge](https://img.shields.io/badge/cartesi--rollups-1.0.0-5bd1d7)](https://docs.cartesi.io/cartesi-rollups/)</a>
  <a href="">[![Static Badge](https://img.shields.io/badge/cartesi-cli-0.9.5-blue)](https://docs.sunodo.io/guide/introduction/what-is-sunodo)</a>
  <a href="">[![Static Badge](https://img.shields.io/badge/python-3.11-yellow)](https://www.python.org/)</a>
</div>



The dApp generates a [logistic regression](https://en.wikipedia.org/wiki/Logistic_regression) model using [scikit-learn](https://scikit-learn.org/), [NumPy](https://numpy.org/) and [pandas](https://pandas.pydata.org/), and then uses [m2cgen (Model to Code Generator)](https://github.com/BayesWitnesses/m2cgen) to transpile that model into native Python code with no dependencies.
This approach is inspired by [Davis David's Machine Learning tutorial](https://www.freecodecamp.org/news/transform-machine-learning-models-into-native-code-with-zero-dependencies/), and is useful for a Cartesi DApp because it removes the need of porting all those Machine Learning libraries to the Cartesi Machine's RISC-V architecture, making the development process easier and the final back-end code simpler to execute.

The practical goal of this application is to predict a classification based on the [Titanic dataset](https://www.kaggle.com/competitions/titanic/data), which shows characteristics of people onboard the Titanic and whether that person survived the disaster or not.
As such, users can submit inputs describing a person's features to find out if that person is likely to have survived.

The model currently takes into account only three characteristics of a person to predict their survival, even though other attributes are available in the dataset:

1. Age
2. Sex, which can be `male` or `female`
3. Embarked, which corresponds to the port of embarkation and can be `C` (Cherbourg), `Q` (Queenstown), or `S` (Southampton)

As such, inputs to the DApp should be given as a JSON string such as the following:

```json
{ "Age": 37, "Sex": "male", "Embarked": "S" }
```

The predicted classification result will be given as `0` (did not survive) or `1` (did survive).

## Environment Setup - Requirements

To use this dApp, you need to set up your environment for Cartesi Rollups development with Cartesi CLI. Below is a link to guide you through the environment setup:

1. [Installation guide](https://docs.cartesi.io/cartesi-rollups/1.3/development/installation/)



## Building and running the application

To build this application, you should have [Cartesi CLI](https://docs.cartesi.io/cartesi-rollups/1.3/development/installation/) installed. With that, simply clone this repository and in the root folder open a terminal window and run:

```shell
cartesi build
```
Give it some minutes to build the docker containers. After that, run :

```shell
cartesi run
```

After running the containers, your terminal window will be similar to the below:

```shell
Attaching to f10a55b8-prompt-1, f10a55b8-validator-1
f10a55b8-prompt-1     | Anvil running at http://localhost:8545
f10a55b8-prompt-1     | GraphQL running at http://localhost:8080/graphql
f10a55b8-prompt-1     | Inspect running at http://localhost:8080/inspect/
f10a55b8-prompt-1     | Explorer running at http://localhost:8080/explorer/
f10a55b8-prompt-1     | Press Ctrl+C to stop the node
```

## Interacting with the application

We can use the `cartesi send generic` command application to interact with the DApp.

First, go to a separate terminal window and send an input. Your terminal window will be similar to below:

```shell
? Chain (Use arrow keys)
❯ Foundry
? Chain Foundry
? RPC URL http://127.0.0.1:8545
? Wallet Mnemonic
? Mnemonic test test test test test test test test test test test junk
? Account 0x3C44CdDdB6a900fa2b585dd299e03d12FA4293BC 10000 ETH
? DApp address 0x70ac08179605AF2D9e75782b8DEcDD3c22aA4D0C
? Input String encoding
? Input (as string) { "Age": 37, "Sex": "male", "Embarked": "S" }
✔ Input sent: 0x02b7506ef56d5560fcc34f09af7de299275a54e42a157b26d1911f7b4b83e09c
```

And your running backend window will be similar to below:

```shell
f10a55b8-validator-1  | [INFO  rollup_http_server::http_service] Received new request of type ADVANCE
f10a55b8-validator-1  | [INFO  actix_web::middleware::logger] 127.0.0.1 "POST /finish HTTP/1.1" 200 290 "-" "python-requests/2.31.0" 0.001344
f10a55b8-validator-1  | INFO:__main__:Received finish status 200
f10a55b8-validator-1  | INFO:__main__:Received advance request data {'metadata': {'msg_sender': '0x3c44cdddb6a900fa2b585dd299e03d12fa4293bc', 'epoch_index': 0, 'input_index': 0, 'block_number': 12, 'timestamp': 1704202333}, 'payload': '0x7b2022416765223a2033372c2022536578223a20226d616c65222c2022456d6261726b6564223a20225322207d'}
f10a55b8-validator-1  | INFO:__main__:Received input: '{ "Age": 37, "Sex": "male", "Embarked": "S" }'
f10a55b8-validator-1  | INFO:__main__:Data={ "Age": 37, "Sex": "male", "Embarked": "S" }, Predicted: 0
f10a55b8-validator-1  | INFO:__main__:Adding notice with payload: 0
f10a55b8-validator-1  | [INFO  actix_web::middleware::logger] 127.0.0.1 "POST /notice HTTP/1.1" 201 11 "-" "python-requests/2.31.0" 0.000896
f10a55b8-validator-1  | INFO:__main__:Received notice status 201 body b'{"index":0}'
f10a55b8-validator-1  | INFO:__main__:Sending finish
```

The payload of the notice corresponds to whether the person survived (`"1"`) or not (`"0"`).
In this case, it should be `"0"`.


## Changing the application

This dApp was implemented in a rather generic way and, as such, it is possible to easily change the target dataset as well as the predictor algorithm.

To change those, open the file `m2cgen/model/build_model.py` and change the following variables defined at the beginning of the script:

- `model`: defines the scikit-learn predictor algorithm to use. While it currently uses `sklearn.linear_model.LogisticRegression`, many [other possibilities](https://scikit-learn.org/stable/modules/classes.html) are available, from several types of linear regressions to solutions such as support vector machines (SVMs).
- `train_csv`: a URL or file path to a CSV file containing the dataset. It should contain a first row with the feature names, followed by the data.
- `include`: an optional list indicating a subset of the dataset's features to be used in the prediction model.
- `dependent_var`: the feature to be predicted, such as the entry's classification