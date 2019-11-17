# convector-example-supplychain-master

## Introduction

This project is an end to end example of a generic supply chain process blockchain based, using the Convector Framework (https://worldsibu.github.io/convector/)

As stated on the site:

*"Convector is a JavaScript-based Development Framework for Enterprise Smart Contract Systems. Its goal is to make it easier for developers to create, test and deploy enterprise-grade DApps by abstracting complexities that make it hard to get started, plus a collection of tools that speed up your go-to-market."*

The goal of this project is to explain the process for creating a working end to end and to  be a baseline for discussion about the best practices of usage of this promising framework.

## Use Case

The use case is a simple representation of a generic supply chain process where the lifecycle of a **Product** is tracked from the extraction of the raw material to make it, until its selling by the **Retailers** .

The entities that participate in this project are:

+ **Supplier**
	+ fetches the raw material that can be supplied to the **Manufacturers**
+ **Manufacturer**
	+ gets the raw material from the **Suppliers**
	+ creates the **Products**
	+ sends the **Products** to the **Distributors**
+ **Distributor**
	+ ships the **Products** to the **Retailers**
+ **Retailer**
	+ orders the **Products** from the **Distributors**
	+ sends the acknowledgement of the reception of the **Products** to the **Distributors**
+ **Customer**
	+ buys the **Products** from the **Retailers**

The high level sequence is described below:
![Supply Chain](https://github.com/xcottos/convector-example-supplychain-master/blob/master/images/supply-chain.bpmn.png "Supply Chain")

## Implementation

This example starts from the installation of the following pre-requisites (there are also the exact versions I'm actually using):

+ Node version 8.15.0
+ Docker 18.09.2
+ npm 6.4.1
+ nvm 0.33.0
+ npx 10.2.0
+ npm-run-all 4.1.5
+ lerna 3.13.0

The following paragraphs describe the procedure to generate this  project from the end to end.

### Project Init

First, the convector-cli package must be installed globally:

**Attention!**: There just was an update of **@worldsibu/convector** to the **1.3** version and the **@worldsibu/convector-cli** to the **1.1.2** version. **This document won't work with earlier versions**.

``
npm i -g @worldsibu/convector-cli
``

Second, hurley package must be installed globally, it will provide an easy way to setup an Hyperledger Fabric 1.4 network:

``
npm i -g @worldsibu/hurley
``

Then we can start creating the project skeleton using the `conv` command that is invoked with 3 parameters:
+ **new** - for generating a new project
+ **supplychain** - is the name of the project and of the root folder  that will be created
+ **-c supplychainchaincode**: default chaincode name

```conv new supplychain -c supplychainchaincode```

With the execution of this script a directory called **supplychain** has been created.

Login into that folder

``cd supplychain``

Into this folder there are some configuration files and the folder **packages**.

Now in the file called **package.json** you have to add a line that will be the definition of the command **cc:invoke** that will be used later to interact with the application

the line to add is the following:

```
"cc:invoke": "f() { chaincode-manager --config ./$2.$1.config.json --user $3 invoke $1 ${@:4}; }; f"
```
so at the end your file should look like

```javascript
{
  "name": "supplychain",
  "version": "0.2.0",
  "description": "Bootstrap project for Chaincodes testNew",
  "files": [
    "dist/*"
  ],
  "scripts": {
    "install": "npm-run-all -s lerna:install",
    "build": "node ./update-paths.js",
    "env:restart": "hurl new -p $PWD/fabric-hurl",
    "test": "npm-run-all -s lerna:test",
    "env:clean": "hurl clean -p $PWD/fabric-hurl",
    "cc:start": "f() { npm run cc:package -- $1 org1; npm run cc:install $1; }; f",
    "cc:upgrade": "f() { npm run cc:package -- $1 org1; hurl upgrade $1 node $2  -P ./chaincode-$1 -p $PWD/fabric-hurl; }; f",
    "===================INTERNALS===================": "===================NO NEED TO CALL THEM DIRECTLY===================",
    "lerna:install": "lerna bootstrap",
    "lerna:build": "lerna run build",
    "cc:package": "f() { npm run lerna:build; chaincode-manager --config ./$2.$1.config.json --output ./chaincode-$1 package; }; f",
    "cc:install": "f() { hurl install $1 node -P ./chaincode-$1 -p $PWD/fabric-hurl; }; f",
    "cc:invoke": "f() { chaincode-manager --config ./$2.$1.config.json --user $3 invoke $1 ${@:4}; }; f",
    "lerna:test": "lerna exec npm run test"
  },
  "devDependencies": {
    "lerna": "^3.13.0",
    "@worldsibu/convector-adapter-mock": "~1.3.0",
    "@worldsibu/convector-platform-fabric": "~1.3.0",
    "@worldsibu/hurley": "~1.0.0",
    "fabric-ca-client": "~1.4.0",
    "fabric-client": "~1.4.0",
    "npm-run-all": "^4.1.5"
  }
}
```

You can notice that differently by the autogenerated package.json here all the **hurl** commands have the parameter **-p $PWD/fabric-hurl**. This is because I personally prefer to have the Hyperledger Fabric environment in the same folder of the project. If you don't put that parameter all the containers and confs will be put in **~/hyperledger-fabric-network**

now install all the packages executing the ``npm i`` command

At the end of the installation you should receive a message like (the numbers may be different):

```
added 904 packages from 567 contributors and audited 42833 packages in 65.967s
found 30 vulnerabilities (10 low, 10 moderate, 10 high)
  run `npm audit fix` to fix them, or `npm audit` for details
```

**Do not run ``npm audit fix`` because it will probably break the installation.**

However this is something I will go in deep in the future.

The chaincode will be deployed on a Hyperledger Fabric 1.4 network defined and ran by **hurley** whose specific folder is called **fabric-hurl** and is located in the home directory of this project; the **hurl** command is executed by the scripts defined in the **package.json** file. Anyway since the **fabric-client** and **fabric-ca-client** are peer dependencies the code can be ran on existing instances without problems. (I will update this project with that part in the future)

Then to check if the skeleton is working:

``npm run env:restart``

The first execution of this command will take some time since it will download the docker images that will be used.

At the end you should receive something like:

```
[hurley] - Ran network restart script
[hurley] - ************ Success!
[hurley] - Complete network deployed at /Users/luca/Projects/GitHubProjects/cloned/convector-example-supplychain-master/fabric-hurl
[hurley] - Setup:
        - Organizations: 2
            * org1
            * org2
        - Users per organization: user1
            * admin
            * user1
        - Channels deployed: 1
            * ch1

[hurley] - You can find the network topology (ports, names) here: /Users/luca/Projects/GitHubProjects/cloned/convector-example-supplychain-master/fabric-hurl/docker-compose.yaml
```
What happened during the execution of the command is that the **Hyperledger Fabric 1.4.0** infrastructure has been started

running the ``docker ps -a`` command:

```
CONTAINER ID        IMAGE                                                                                                                 COMMAND                  CREATED             STATUS              PORTS                                                                    NAMES
9689cb760462        hyperledger/fabric-peer:1.4.0                                                                                         "peer node start --p…"   About an hour ago   Up About an hour    0.0.0.0:7051-7053->7051-7053/tcp                                         peer0.org1.hurley.lab
7333bb820374        hyperledger/fabric-peer:1.4.0                                                                                         "peer node start --p…"   About an hour ago   Up About an hour    0.0.0.0:7151->7051/tcp, 0.0.0.0:7152->7052/tcp, 0.0.0.0:7153->7053/tcp   peer0.org2.hurley.lab
6c07fed50b69        hyperledger/fabric-ca:1.4.0                                                                                           "fabric-ca-server st…"   About an hour ago   Up About an hour    0.0.0.0:7154->7054/tcp                                                   ca.org2.hurley.lab
382b08596a70        hyperledger/fabric-ca:1.4.0                                                                                           "fabric-ca-server st…"   About an hour ago   Up About an hour    0.0.0.0:7054->7054/tcp                                                   ca.org1.hurley.lab
2f721919bd7e        hyperledger/fabric-couchdb:0.4.14                                                                                     "tini -- /docker-ent…"   About an hour ago   Up About an hour    4369/tcp, 9100/tcp, 0.0.0.0:5184->5984/tcp                               couchdb.peer0.org2.hurley.lab
abe4eadac764        hyperledger/fabric-couchdb:0.4.14                                                                                     "tini -- /docker-ent…"   About an hour ago   Up About an hour    4369/tcp, 9100/tcp, 0.0.0.0:5084->5984/tcp                               couchdb.peer0.org1.hurley.lab
2bb40eec25e7        hyperledger/fabric-orderer:1.4.0                                                                                      "orderer"                About an hour ago   Up About an hour    0.0.0.0:7050->7050/tcp                                                   orderer.hurley.lab
```

We can see there is:

+ 1 Orderer (orderer.example.com)
+ 2 Certificate Authorities (ca.org1.example.com, ca.org2.example.com)
+ 2 Peers (peer0.org1.example.com, peer0.org2.example.com)
+ 2 CouchDB instances (couchdb.peer0.org1.hurley.lab, couchdb.peer0.org2.hurley.lab)

Now some users in the organizations (**org1** and **org2**) have been registered via the certificate authorities  we'll use them for interacting with the system; if there are no errors on screen it should be all ok.

### Model/Controller pattern

The Convector framework approach follows the Model/Controller pattern that is well explained on the site:

_"Convector is designed to help you write reusable pieces of code that describe the nature of what a developer can do in a blockchain. A blockchain, in the developer’s eyes, is no more than a **data layer** protected by a **logic layer** defining the rules of what the outside world can do in with the inner data. Thus, a really comfortable way of writing chaincode logic (smart contracts) is by having **Models** describing the shape of the data and **Controllers** describing the actions and rules that apply to the models."_

so we re going to define the **Models** and the **Controllers**

First ``cd packages/supplychainchaincode-cc/src`` where the chaincode **supplychainchaincode** code is located.

You will see that 2 files have been created:

+ supplychainchaincode.model.ts (a sample Model)
+ supplychainchaincode.controller.ts (a sample Controller)


#### Models

with the ``conv generate`` command we now generate the stubs of our Models.

Now cd back to the root directory and run:

```
conv generate model Supplier -c supplychainchaincode
conv generate model Manufacturer -c supplychainchaincode
conv generate model Distributor -c supplychainchaincode
conv generate model Retailer -c supplychainchaincode
conv generate model Customer -c supplychainchaincode
```
You will see that now in the directory ``cd packages/supplychainchaincode-cc/src`` a file for each Model has been created:

```
Customer.model.ts
Distributor.model.ts
Manufacturer.model.ts
Retailer.model.ts
Supplier.model.ts
```

**NOTE:** The **@worldsibu/convector-cli** now changes the case of the model name to  **Camel** case, so the files could be named like **customer.model.ts**. What's important is to remember to update the **index.ts** accordingly if changing the generated names.

These files are written using Typescript language ([https://www.typescriptlang.org](https://www.typescriptlang.org)) that is expressive and generic enough for being used as a base of code generation (in this case the javascript code)

**NOTE:** In the github project you can notice I moved the models into a **models** folder. This is up to you, just remember to update the **index.ts** accordingly.

Opening each of them (i.e. Supplier.model.ts), you will see that there are also the properties **created** and **modified**. I decided to comment them out and to deal with them in the future.

Same thing has to be done to all the Models.

Now we need to add the variables that are specific for every Model:

+ **Supplier**
	+ rawMaterialAvailable: is a number that expresses the quantity of raw material available to be supplied
+ **Manufacturer**
	+ rawMaterialAvailable: is a number that expresses the quantity of raw material available to be used for creating products
	+ productsAvailable: is a number that expresses the quantity of products ready to be distributed
+ **Distributor**
	+ productsToBeShipped: is a number that expresses the quantity of products ready to be shipped
	+ productsShipped: is a number that expresses the quantity of products shipped
	+ productsReceived: is a number that expresses the quantity of products shipped that have been received.
+ **Retailer**
	+ productsOrdered: is a number that expresses the quantity of products ordered
	+ productsAvailable: is a number that expresses the quantity of products available for being sold
	+ productsSold: is a number that expresses the quantity of products that have been sold
+ **Customer**
	+ productsBought: is a number that expresses the quantity of products bought

So the **Supplier.model.ts** shoud now look like:
```javascript
import * as yup from 'yup';
import {
  ConvectorModel,
  Default,
  ReadOnly,
  Required,
  Validate
} from '@worldsibu/convector-core-model';

export class Supplier extends ConvectorModel<Supplier> {
  @ReadOnly()
  @Required()
  public readonly type = 'io.worldsibu.Supplier';

  @Required()
  @Validate(yup.string())
  public name: string;

  // @ReadOnly()
  // @Required()
  // @Validate(yup.number())
  // public created: number;
  //
  // @Required()
  // @Validate(yup.number())
  // public modified: number;

  @Required()
  @Validate(yup.number())
  public rawMaterialAvailable: number;

}
```

The **Manufacturer.model.ts** shoud now look like:
```javascript
import * as yup from 'yup';
import {
  ConvectorModel,
  Default,
  ReadOnly,
  Required,
  Validate
} from '@worldsibu/convector-core-model';

export class Manufacturer extends ConvectorModel<Manufacturer> {
  @ReadOnly()
  @Required()
  public readonly type = 'io.worldsibu.Manufacturer';

  @Required()
  @Validate(yup.string())
  public name: string;

  // @ReadOnly()
  // @Required()
  // @Validate(yup.number())
  // public created: number;
  //
  // @Required()
  // @Validate(yup.number())
  // public modified: number;

  @Required()
  @Validate(yup.number())
  public productsAvailable: number;

  @Required()
  @Validate(yup.number())
  public rawMaterialAvailable: number;

}
```
The **Distributor.model.ts** shoud now look like:
```javascript
import * as yup from 'yup';
import {
  ConvectorModel,
  Default,
  ReadOnly,
  Required,
  Validate
} from '@worldsibu/convector-core-model';

export class Distributor extends ConvectorModel<Distributor> {
  @ReadOnly()
  @Required()
  public readonly type = 'io.worldsibu.Distributor';

  @Required()
  @Validate(yup.string())
  public name: string;

  // @ReadOnly()
  // @Required()
  // @Validate(yup.number())
  // public created: number;
  //
  // @Required()
  // @Validate(yup.number())
  // public modified: number;

  @Required()
  @Validate(yup.number())
  public productsToBeShipped: number;

  @Required()
  @Validate(yup.number())
  public productsShipped: number;

  @Required()
  @Validate(yup.number())
  public productsReceived: number;
}

```
The **Retailer.model.ts** shoud now look like:
```javascript
import * as yup from 'yup';
import {
  ConvectorModel,
  Default,
  ReadOnly,
  Required,
  Validate
} from '@worldsibu/convector-core-model';

export class Retailer extends ConvectorModel<Retailer> {
  @ReadOnly()
  @Required()
  public readonly type = 'io.worldsibu.Retailer';

  @Required()
  @Validate(yup.string())
  public name: string;

  // @ReadOnly()
  // @Required()
  // @Validate(yup.number())
  // public created: number;
  //
  // @Required()
  // @Validate(yup.number())
  // public modified: number;

  @Required()
  @Validate(yup.number())
  public productsOrdered: number;

  @Required()
  @Validate(yup.number())
  public productsAvailable: number;

  @Required()
  @Validate(yup.number())
  public productsSold: number;
}
```
The **Customer.model.ts** shoud now look like:
```javascript
import * as yup from 'yup';
import {
  ConvectorModel,
  Default,
  ReadOnly,
  Required,
  Validate
} from '@worldsibu/convector-core-model';

export class Customer extends ConvectorModel<Customer> {
  @ReadOnly()
  @Required()
  public readonly type = 'io.worldsibu.Customer';

  @Required()
  @Validate(yup.string())
  public name: string;

  // @ReadOnly()
  // @Required()
  // @Validate(yup.number())
  // public created: number;
  //
  // @Required()
  // @Validate(yup.number())
  // public modified: number;

  @Required()
  @Validate(yup.number())
  public productsBought: number;
}
```
You can notice that there are no validations but the type ones. I will take care of refining it in the future.

Now that the Models have been created it's time to implement the logic.

#### Controller
For the Controller I will modify directly on the file called **supplychainchaincode.controller.ts** to keep the standard naming of the Controller in this folder

The Controller will contain all the logic for implementing the actions described in the **Use Case** section; specifically will contain the implementation of all the following **functions** that implement the logic:

**fetchRawMaterial:**
![Fetch Raw Material](https://github.com/xcottos/convector-example-supplychain-master/blob/master/images/fetchRawMaterial.png "Fetch Raw Material")

**getRawMaterialFromSupplier:**
![Get Raw Material](https://github.com/xcottos/convector-example-supplychain-master/blob/master/images/getRawMaterial.png "Get Raw Material")

**createProducts:**
![Create Products](https://github.com/xcottos/convector-example-supplychain-master/blob/master/images/createProducts.png "Create Products")

**sendProductsToDistribution:**
![Send Products to Distribution](https://github.com/xcottos/convector-example-supplychain-master/blob/master/images/sendsProductsToDistribution.png "Send Products to Distribution")

**orderProductsFromDistributor:**
![Order Products](https://github.com/xcottos/convector-example-supplychain-master/blob/master/images/orderShipProducts.png "Order Products")

**receiveProductsFromDistributor:**
![Receive Products](https://github.com/xcottos/convector-example-supplychain-master/blob/master/images/receiveProducts.png "Receive Products")

**buyProductsFromRetailer:**
![Buy Products](https://github.com/xcottos/convector-example-supplychain-master/blob/master/images/buyProducts.png "Buy Products")

Together with these functions I created also others that are used as helpers:

+ **createSupplier**: creates a Supplier
+ **createManufacturer**: creates a Manufacturer
+ **createDistributor**: creates a Distributor
+ **createRetailer**: creates a Retailer
+ **createCustomer**: creates a Customer
+ **getAllSuppliers**: shows all the created Suppliers
+ **getAllManufacturers**: shows all the created Manufacturers
+ **getAllDistributors**: shows all the created Distributors
+ **getAllRetailers**: shows all the created Retailers
+ **getAllCustomers**: shows all the created Customers
+ **getAllModels**: shows all the created Models

The implementation of the Controller is quite straight forward once there are few concepts clear:

Since in the Controller we manage Models the first step is understanding more in deep what a Model is as class: every Model extends a class called ConvectorModel&lt;T&gt; and you can read its definition in the file **https://github.com/hyperledger-labs/convector/blob/develop/%40worldsibu/convector-core-model/src/convector-model.ts**

```javascript
/** @module convector-core-model */

import * as yup from 'yup';
import { InvalidIdError } from '@worldsibu/convector-core-errors';
import { BaseStorage } from '@worldsibu/convector-core-storage';

import { Validate } from '../src/validate.decorator';
import { getDefaults } from '../src/default.decorator';
import { Required, ensureRequired } from '../src/required.decorator';
import {
  getPropertiesValidation,
  getValidatedProperties
} from '../src/validate.decorator';

export type RequiredKeys<T> = { [K in keyof T]-?:
  string extends K ? never : number extends K ? never : {} extends Pick<T, K> ? never : K
} extends { [_ in keyof T]-?: infer U } ? U extends keyof T ? U : never : never;

export type OptionalKeys<T> = { [K in keyof T]-?:
  string extends K ? never : number extends K ? never : {} extends Pick<T, K> ? K : never
} extends { [_ in keyof T]-?: infer U } ? U extends keyof T ? U : never : never;

export declare type FlatConvectorModel<T> =
  {[L in Exclude<OptionalKeys<T>, keyof ConvectorModel<any>>]?: T[L]} &
  {[L in Exclude<RequiredKeys<T>, keyof ConvectorModel<any>>]: T[L]};

export interface History<T> {
  value: T;
  txId: string;
  timestamp: number;
}

/**
 * This class is intended to be inherited by all the models of the application.
 *
 * It provides the underlying communication with the [[BaseStorage]].
 */
export abstract class ConvectorModel<T extends ConvectorModel<any>> {
  private static type = 'io.convector.model';

  public static schema<T extends ConvectorModel<any>>(
    this: Function&{prototype: T}
  ): yup.ObjectSchema<FlatConvectorModel<T>&{id:string,type:string}> {
    return yup.object<FlatConvectorModel<T>&{id:string,type:string}>().shape({
      id: yup.string().required(),
      type: yup.string(),
      ...getPropertiesValidation(this.prototype)
    } as any);
  }

  /**
   * Fetch one model by its id and instantiate the result
   *
   * @param this The extender type
   * @param id The ID used to fetch the model
   * @param type The type to use for instantiation, if not provided, the extender type is used
   */
  public static async getOne<T extends ConvectorModel<any>>(
    this: new (content: any) => T,
    id: string,
    type?: new (content: any) => T,
    storageOptions?: any
  ): Promise<T> {
    type = type || this;
    const content = await BaseStorage.current.get(id, storageOptions);

    const model = new type(content);

    if ((content && model) && content.type !== model.type) {
      throw new Error(`Possible ID collision, element ${id} of type ${content.type} is not ${model.type}`);
    }

    return model;
  }

  /**
   * Runs a query on the storage layer
   *
   * @param this The extender type
   * @param type The type to use for instantiation, if not provided, the extender type is used
   * @param args The query params, this is passed directly to the current storage being used
   */
  public static async query<T>(type: new (content: any) => T, ...args: any[]): Promise<T | T[]>;
  public static async query<T>(this: new (content: any) => T, ...args: any[]): Promise<T | T[]> {
    let type = this;

    // Stupid horrible hack to find the current implementation's parent type
    if (args[0] && 'type' in args[0] && args[0].type === ConvectorModel.type) {
      type = args.shift();
    }

    const content = await BaseStorage.current.query(...args);
    return Array.isArray(content) ? content.map(c => new type(c)) : new type(content);
  }

  /**
   * Return all the models with the given [[ConvectorModel.type]]
   *
   * @param this The extender type
   * @param type The type field to lookup and group the results
   */
  public static async getAll<T extends ConvectorModel<any>>(
    this: new (content: any) => T,
    type?: string
  ): Promise<T[]> {
    type = type || new this('').type;
    return await ConvectorModel.query(this, { selector: { type } }) as T[];
  }

  /**
   * This field is [[Required]] and [[Validate]]d using a string schema
   *
   * Represents the key used to store the model in the blockchain
   */
  @Required()
  @Validate(yup.string())
  public id: string;

  /**
   * This field must be provided by the extender class.
   *
   * It should be [[Required]] and [[ReadOnly]]
   *
   * We normally use a domain name patter for type names, i.e.: `io.worldsibu.example.user`
   */
  public abstract readonly type: string;

  /**
   * The constructor can be called in multiple ways.
   *
   * - As an empty box where you instantiate one and start adding data
   * - As a data fetcher, providing the ID and using [[ConvectorModel.fetch]]
   * - As a formatter, you just pass an object of any shape into the constructor
   *    and it will trim the remaining content and leave what's important
   */
  constructor();
  constructor(id: string);
  constructor(content: { [key in keyof T]?: T[key] });
  constructor(content?: string | { [key in keyof T]?: T[key] }) {
    if (!content) {
      return;
    }

    if (typeof content === 'string') {
      this.id = content;
      return;
    }

    this.assign(content);
  }

  /**
   * Given one model loaded into the instance,
   * update its content with the object passed in the param.
   *
   * Store the result after the update occurs.
   */
  public async update(content: { [key in keyof T]?: T[key] }) {
    this.assign(content);
    await this.save();
  }

  /**
   * Invokes the [[BaseStorage.get]] method to retrieve the model from storage.
   */
  public async fetch(storageOptions?: any) {
    const content = await BaseStorage.current.get(this.id, storageOptions) as ConvectorModel<T>;

    if (content.type !== this.type) {
      throw new Error(`Possible ID collision, element ${this.id} of type ${content.type} is not ${this.type}`);
    }

    this.assign(content as T);
  }

  public async history(): Promise<History<T>[]> {
    const history = await BaseStorage.current.history(this.id);

    return history.map(item => ({
      txId: item.tx_id,
      value: new (this.constructor as new (content: any) => T)(item.value),
      timestamp: item.timestamp
    }));
  }

  /**
   * Invokes the [[BaseStorage.set]] method to write into chaincode.
   *
   * @param storageOptions Extra options to pass to the storage layer. The type depends on the storage
   */
  public async save(storageOptions?: any) {
    this.assign(getDefaults(this), true);
    if (!ensureRequired(this)) {
      if (!this.id) {
        throw new Error(`Model ${this.type} is missing the 'id' property \n${JSON.stringify(this)}`);
      } else {
        throw new Error(`Model ${this.type} is not complete\n${JSON.stringify(this)}.
        Check your model definition for more details.`);
      }
    }

    InvalidIdError.test(this.id);
    await BaseStorage.current.set(this.id, this, storageOptions);
  }

  /**
   * Make a copy of this model
   */
  public clone(): T {
    return new (this.constructor as new (content: any) => T)(Object.assign({}, this));
  }

  /**
   * Serealize this model so it can be transferred in the network
   *
   * @param skipEmpty Skip the empty properties
   */
  public toJSON(skipEmpty = false): { [key in keyof T]?: T[key] } {
    let protos = [];
    let children = this;

    // Search through the parents until reach ConvectorModel
    do {
      children = Object.getPrototypeOf(children);
      protos.push(children);
    } while (children['__proto__'].constructor.name !== ConvectorModel.name);

    // Get all the descriptors for this model
    const descriptors = protos.reduce((result, proto) => [
      ...result,
      ...Object.keys(proto)
        // Map the keys to their [key,propDescriptior]
        .map(key => [key, Object.getOwnPropertyDescriptor(proto, key)])
    ], []);

    // debugger;
    const base = Object.keys(this).concat('id')
      .filter(k => !k.startsWith('_'))
      .filter(k => !skipEmpty || !(this[k] === undefined || this[k] === null))
      .reduce((result, key) => ({ ...result, [key]: this[key] }), {});

    return descriptors
      .reduce((result, [key, desc]) => {
        const hasGetter = desc && typeof desc.get === 'function';

        if (hasGetter) {
          // Apply the descriptors to the children class
          result[key] = desc.get.call(this);
        }

        if (skipEmpty && (result[key] === undefined || result[key] === null)) {
          delete result[key];
        }

        if (result[key] instanceof ConvectorModel) {
          result[key] = result[key].toJSON(true);
        }

        return result;
      }, base);
  }

  /**
   * Delete a model and persist the changes into the blockchain.
   *
   * Notice that there's no such a concept as **delete** in the blockchain,
   * so what this does is to remove all the reachable references to the model.
   */
  public async delete(storageOptions?: any) {
    await BaseStorage.current.delete(this.id, storageOptions);
  }

  /**
   * Extend the current model definition with some more data
   *
   * @hidden
   *
   * @param defaults Should the [[Default]]s be applied
   */
  private assign(content: { [key in keyof T]?: T[key] }, defaults = false) {
    const validated = ['id', 'type', ...getValidatedProperties(this)];
    const filteredContent = Object.keys(content)
      .map(key => key.replace(/^_/, ''))
      .filter(key => validated.indexOf(key) >= 0)
      .reduce((result, key) => ({
        ...result,
        [key]: content[key] !== undefined ? content[key] : content['_' + key]
      }), {});

    const afterDefaults = defaults ? this.toJSON(true) : {};
    Object.assign(this, filteredContent, afterDefaults);
  }
}
```
Here you can see that every Model, extending this class, inherited various methods:
+ some static ones: like **getOne**, **getAll** etc
+ some instance ones: like **save**, **update** and **delete**

And the field **id** that **must be set** when an instance of a Model is created.

If we now we analyze one of the methods in the Controller for creating a Model we can see already the usage of some of them:

**createSupplier**
```javascript
@Invokable()
public async createSupplier(
	@Param(Supplier)
	supplier: Supplier
) {
	await supplier.save();
}
```
The method is really straight forward because:
+ takes as input a Supplier object
+ uses the **save()** instance method for saving in the ledger the instance of the Supplier

All these methods follow the **async/await** pattern to be synchronous.

Another example to be explained is a function that retrieves all the instances of a specific Model, like the **getAllSuppliers** :

```javascript
@Invokable()
public async getAllSuppliers()
{
	const storedSuppliers = await Supplier.getAll<Supplier>();
	return storedSuppliers;
}
```

This method uses the **getAll** method that needs to define the generic after its name to tell the adapter which Model type we want to retrieve all the instances. In this case the generic is **Supplier** since we want to retrieve all the Suppliers in the ledger.

Another example to be explained is a function that impacts on Models that have been already stored in the ledger, like the **getRawMaterialFromSupplier** that is used to transfer raw material from the Supplier to the Manufacturer:

```javascript
@Invokable()
public async getRawMaterialFromSupplier(
	@Param(yup.string())
	manufacturerId: string,
	@Param(yup.string())
	supplierId: string,
	@Param(yup.number())
	rawMaterialSupply: number
) {
	const supplier = await Supplier.getOne(supplierId);
	supplier.rawMaterialAvailable = supplier.rawMaterialAvailable - rawMaterialSupply;
	const manufacturer = await Manufacturer.getOne(manufacturerId);
	manufacturer.rawMaterialAvailable = rawMaterialSupply + manufacturer.rawMaterialAvailable;

	await supplier.save();
	await manufacturer.save();
}
```
This function:
+ takes as input the ids of the Manufacturer and the Supplier and a number that represents the quantity of raw material to be tranfered
+ uses the static method **getOne** to retrieve the instances of Supplier and Manufacturer that have the id passed as parameters
+ changes the value of the variables in the instances of this 2 Models
+ save the models with the **save** function

All the functions are using these methods.

The file **supplychainchaincode.controller.ts** should look like this:
```javascript
import * as yup from 'yup';
import {
  Controller,
  ConvectorController,
  Invokable,
  Param
} from '@worldsibu/convector-core-controller';

import { Supplier } from './Supplier.model';
import { Manufacturer } from './Manufacturer.model';
import { Distributor } from './Distributor.model';
import { Retailer } from './Retailer.model';
import { Customer } from './Customer.model';

@Controller('supplychainchaincode')
export class SupplychainchaincodeController extends ConvectorController {

  @Invokable()
  public async createSupplier(
    @Param(Supplier)
    supplier: Supplier
  ) {
    await supplier.save();
  }

  @Invokable()
  public async createManufacturer(
    @Param(Manufacturer)
    manufacturer: Manufacturer
  ) {
    await manufacturer.save();
  }

  @Invokable()
  public async createDistributor(
    @Param(Distributor)
    distributor: Distributor
  ) {
    await distributor.save();
  }

  @Invokable()
  public async createRetailer(
    @Param(Retailer)
    retailer: Retailer
  ) {
    await retailer.save();
  }

  @Invokable()
  public async createCustomer(
    @Param(Customer)
    customer: Customer
  ) {
    await customer.save();
  }

  @Invokable()
  public async getAllSuppliers()
  {
    const storedSuppliers = await Supplier.getAll<Supplier>();
    return storedSuppliers;
  }

  @Invokable()
  public async getSupplierById(
    @Param(yup.string())
    supplierId: string
  )
  {
    const supplier = await Supplier.getOne(supplierId);
    return supplier;
  }

  @Invokable()
  public async getAllManufacturers()
  {
    const storedManufacturers = await Manufacturer.getAll<Manufacturer>();
		return storedManufacturers;
  }

  @Invokable()
  public async getManufacturerById(
    @Param(yup.string())
    manufacturerId: string
  )
  {
    const manufacturer = await Manufacturer.getOne(manufacturerId);
    return manufacturer;
  }

  @Invokable()
  public async getAllDistributors()
  {
    const storedDistributors = await Distributor.getAll<Distributor>();
    return storedDistributors
  }

  @Invokable()
  public async getDistributorById(
    @Param(yup.string())
    distributorId: string
  )
  {
    const distributor = await Distributor.getOne(distributorId);
    return distributor;
  }

  @Invokable()
  public async getAllRetailers()
  {
    const storedRetailers = await Retailer.getAll<Retailer>();
    return storedRetailers;
  }

  @Invokable()
  public async getRetailerById(
    @Param(yup.string())
    retailerId: string
  )
  {
    const retailer = await Retailer.getOne(retailerId);
    return retailer;
  }

  @Invokable()
  public async getAllCustomers()
  {
    const storedCustomers = await Customer.getAll<Customer>();
    return storedCustomers;
  }

  @Invokable()
  public async getCustomerById(
    @Param(yup.string())
    customerId: string
  )
  {
    const customer = await Customer.getOne(customerId);
    return customer;
  }

  @Invokable()
  public async getAllModels()
  {
    const storedCustomers = await Customer.getAll<Customer>();
    console.log(storedCustomers);

    const storedRetailers = await Retailer.getAll<Retailer>();
    console.log(storedRetailers);

    const storedDistributors = await Distributor.getAll<Distributor>();
    console.log(storedDistributors);

    const storedManufacturers = await Manufacturer.getAll<Manufacturer>();
    console.log(storedManufacturers);

    const storedSuppliers = await Supplier.getAll<Supplier>();
    console.log(storedSuppliers);
  }

  @Invokable()
  public async fetchRawMaterial(
    @Param(yup.string())
    supplierId: string,
    @Param(yup.number())
    rawMaterialSupply: number
  ) {
    const supplier = await Supplier.getOne(supplierId);
    supplier.rawMaterialAvailable = supplier.rawMaterialAvailable + rawMaterialSupply;
    await supplier.save();
  }

  @Invokable()
  public async getRawMaterialFromSupplier(
    @Param(yup.string())
    manufacturerId: string,
    @Param(yup.string())
    supplierId: string,
    @Param(yup.number())
    rawMaterialSupply: number
  ) {
    const supplier = await Supplier.getOne(supplierId);
    supplier.rawMaterialAvailable = supplier.rawMaterialAvailable - rawMaterialSupply;
    const manufacturer = await Manufacturer.getOne(manufacturerId);
    manufacturer.rawMaterialAvailable = rawMaterialSupply + manufacturer.rawMaterialAvailable;

    await supplier.save();
    await manufacturer.save();
  }

  @Invokable()
  public async createProducts(
    @Param(yup.string())
    manufacturerId: string,
    @Param(yup.number())
    rawMaterialConsumed: number,
    @Param(yup.number())
    productsCreated: number
  ) {
    const manufacturer = await Manufacturer.getOne(manufacturerId);
    manufacturer.rawMaterialAvailable = manufacturer.rawMaterialAvailable - rawMaterialConsumed;
    manufacturer.productsAvailable = manufacturer.productsAvailable + productsCreated;
    await manufacturer.save();
  }

  @Invokable()
  public async sendProductsToDistribution(
    @Param(yup.string())
    manufacturerId: string,
    @Param(yup.string())
    distributorId: string,
    @Param(yup.number())
    sentProducts: number
  ) {
    const distributor = await Distributor.getOne(distributorId);
    distributor.productsToBeShipped = distributor.productsToBeShipped + sentProducts;
    const manufacturer = await Manufacturer.getOne(manufacturerId);
    manufacturer.productsAvailable = manufacturer.productsAvailable - sentProducts;

    await distributor.save();
    await manufacturer.save();
  }

  @Invokable()
  public async orderProductsFromDistributor(
    @Param(yup.string())
    retailerId: string,
    @Param(yup.string())
    distributorId: string,
    @Param(yup.number())
    orderedProducts: number
  ) {
    const retailer = await Retailer.getOne(retailerId);
    retailer.productsOrdered = retailer.productsOrdered + orderedProducts;
    const distributor = await Distributor.getOne(distributorId);
    distributor.productsToBeShipped = distributor.productsToBeShipped - orderedProducts;
    distributor.productsShipped = distributor.productsShipped + orderedProducts;

    await retailer.save();
    await distributor.save();
  }

  @Invokable()
  public async receiveProductsFromDistributor(
    @Param(yup.string())
    retailerId: string,
    @Param(yup.string())
    distributorId: string,
    @Param(yup.number())
    receivedProducts: number
  ) {
    const retailer = await Retailer.getOne(retailerId);
    retailer.productsAvailable = retailer.productsAvailable + receivedProducts;
    const distributor = await Distributor.getOne(distributorId);
    distributor.productsReceived = distributor.productsReceived + receivedProducts;

    await retailer.save();
    await distributor.save();
  }

  @Invokable()
  public async buyProductsFromRetailer(
    @Param(yup.string())
    retailerId: string,
    @Param(yup.string())
    customerId: string,
    @Param(yup.number())
    boughtProducts: number
  ) {
    const retailer = await Retailer.getOne(retailerId);
    retailer.productsAvailable = retailer.productsAvailable - boughtProducts;
    retailer.productsSold = retailer.productsSold + boughtProducts;
    const customer = await Customer.getOne(customerId);
    customer.productsBought = customer.productsBought + boughtProducts;

    await retailer.save();
    await customer.save();
  }
}
```

## Installation and execution of the code
Once the Models and Controller have been written, you have to go back to the root folder (**supplychain**) and run  the command ``npm i``

If there are no errors you should see the output that tells you the chaincode called **supplychainchaincode-cc@0.1.0** has been processed (generating the client, the controller interface etc.)

Now it's time to install and start the chaincode with the command ``npm run cc:start`` that is defined in the package.json as:
```JavaScript
"cc:start": "f() { npm run cc:package -- $1 org1; npm run cc:install $1; }; f"
```
So it executes 2 commands passing the **chaincode name** as first parameter (the $1), and the name **org1** as second parameter ($2):

+ cc:package -- $1 org1 (It creates the package to be installed through the command below)
```javascript
"cc:package": "f() { npm run lerna:build; chaincode-manager --config ./$2.$1.config.json --output ./chaincode-$1 package; }; f",
```
+ npm run cc:install $1 (it installs the chaincode using hurl through the command below)
```javascript
"cc:install": "f() { hurl install $1 node -P ./chaincode-$1 -p $PWD/fabric-hurl; }; f",
```

The execution of this command:

```
npm run cc:start -- supplychainchaincode  1
```

should end with an output like the following:

```
Installing Chaincode at org1
2019-02-25 11:52:35.244 CET [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2019-02-25 11:52:35.244 CET [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
2019-02-25 11:52:35.338 CET [chaincodeCmd] install -> INFO 003 Installed remotely response:<status:200 payload:"OK" >
Installed Chaincode at org1
Installing Chaincode at org2
2019-02-25 11:52:35.401 CET [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2019-02-25 11:52:35.401 CET [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
2019-02-25 11:52:35.487 CET [chaincodeCmd] install -> INFO 003 Installed remotely response:<status:200 payload:"OK" >
Installed Chaincode at org2
Instantiating Chaincode at org1
It may take a few minutes depending on the chaincode dependencies
2019-02-25 11:52:45.572 CET [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2019-02-25 11:52:45.572 CET [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
Instantiated Chaincode at org1
```

Running now the command ``docker ps -a`` you should notice that there are 2 new containers:
```
2a488ae54b4d        dev-peer0.org2.hurley.lab-supplychainchaincode-1.0-982db5386d51b5f5bf00ddc0470aff5e11fa77d9a5409fd6991eb7791002a5c3   "/bin/sh -c 'cd /usr…"   About an hour ago   Up About an hour                                                                             dev-peer0.org2.hurley.lab-supplychainchaincode-1.0
807e9ed0d7c9        dev-peer0.org1.hurley.lab-supplychainchaincode-1.0-5aea7809261e79ace2a874a1d67b3a998cccc93ce6142f80105dfa2abaa958f5   "/bin/sh -c 'cd /usr…"   About an hour ago   Up About an hour                                                                             dev-peer0.org1.hurley.lab-supplychainchaincode-1.0
```
That are the 2 containers, one per organization, called ``dev-peer0.org2.hurley.lab-supplychainchaincode-1.0`` and ``dev-peer0.org1.example.com-supplychainchaincode-1.0``  that are running the chaincode.

## Interaction with the chaincode
For interacting with the chaincode we'll use the command ``npm run cc:invoke`` that is defined in the package.json as:
```
"cc:invoke": "f() { chaincode-manager --config ./$2.$1.config.json --user $3 invoke $1 ${@:4}; }; f"
```
It invokes the ``chaincode-manager`` command that is defined in ``node_modules/@worldsibu/convector-tool-chaincode-manager``

It takes as inputs 4 parameters:
+ chaincode name
+ organization name
+ user name
+ controller name


**NOTE FOR LINUX USERS**

You have to force the /bin/bash as the default shell for the execution of the npm run scripts since linux uses sh as default shell for npm run scripts. In order to do that you have to run:

```
npm config set script-shell /bin/bash
```

**NOTE FOR UBUNTU 18.04 USERS**

if you already updated npm to the latest version and npm -v still gives you the wrong version (like 3.5.7) it's because of the cache of bash so you have to:

+ sudo apt purge npm
+ ln -s /usr/local/bin/npm /usr/bin/npm
+ npm config set script-shell /bin/bash

In our case chaincode name and controller name are the same so a sample invocation is:
```
npm run cc:invoke -- supplychainchaincode org1 user1 supplychainchaincode createSupplier '{"id":"SPL_1","name":"supplier1","rawMaterialAvailable":2000}'
```


This will invoke the method called **createSupplier** that we wrote in the **Controller** and will create a **Supplier** with id **SPL_1** that will be saved into the ledger.

In the folder **packages/supplychainchaincode-cc/script/** there's a file called **testScript_sh.bash** that contains a script that:
+ restarts the dev-env environment (the Hyperledger nodes)
+ rebuild and reinstall the chaincode and gives it as version number 1
+ creates a scenario with:
	+ 2 Suppliers (SPL_1 and SPL_2)
	+ 2 Manufacturers (MNF_1 and MNF_2)
	+ 2 Distributors (DST_1 and DST_2)
	+ 2 Retailers (RTL_1, RTL_2)
	+ 3 Customers (CST_1, CST_2 and CST_3)

where these entities interacts from the end to end: from the fetching of raw material until the selling of the products.
+ prints all the Models

This happens running the script **testScript_sh.bash** with the command
```
bash testScript_sh.bash
```
To read the messages written on the console via the invocations of the ``console.log``  function within the Controller, you need to connect to one of the peers that executes the chaincode.

This is done with the command ``docker logs`` that accepts as parameter the id of the container that you saw as part of the output of the ``docker ps -a`` command.

In our scenario we use **2a488ae54b4d** that corresponds to **dev-peer0.org2.example.com-supplychainchaincode-1**

so the command will be:
```
docker logs 2a488ae54b4d -f
```
Running this command you will see all the logs of the container in real time. It will show also all the past logs.

The last command output is the print of all the Models that shows the status of each of them after the execution of the end to end scenario:

```javascript
debug: [Chaincode] ============= START : supplychainchaincode_getAllModels ===========
debug: [StubHelper] Query: {"selector":{"type":"io.worldsibu.Customer"}}
[ Customer {
    _id: 'CST_1',
    _name: 'luca',
    _productsBought: 2,
    _type: 'io.worldsibu.Customer' },
  Customer {
    _id: 'CST_2',
    _name: 'diestrin',
    _productsBought: 4,
    _type: 'io.worldsibu.Customer' },
  Customer {
    _id: 'CST_3',
    _name: 'waltermontes',
    _productsBought: 0,
    _type: 'io.worldsibu.Customer' } ]
debug: [StubHelper] Query: {"selector":{"type":"io.worldsibu.Retailer"}}
[ Retailer {
    _id: 'RTL_1',
    _name: 'retailer1',
    _productsAvailable: 2,
    _productsOrdered: 6,
    _productsSold: 4,
    _type: 'io.worldsibu.Retailer' },
  Retailer {
    _id: 'RTL_2',
    _name: 'retailer2',
    _productsAvailable: 0,
    _productsOrdered: 2,
    _productsSold: 2,
    _type: 'io.worldsibu.Retailer' } ]
debug: [StubHelper] Query: {"selector":{"type":"io.worldsibu.Distributor"}}
[ Distributor {
    _id: 'DST_1',
    _name: 'distributor1',
    _productsReceived: 5,
    _productsShipped: 5,
    _productsToBeShipped: 6,
    _type: 'io.worldsibu.Distributor' },
  Distributor {
    _id: 'DST_2',
    _name: 'distributor2',
    _productsReceived: 3,
    _productsShipped: 3,
    _productsToBeShipped: 0,
    _type: 'io.worldsibu.Distributor' } ]
debug: [StubHelper] Query: {"selector":{"type":"io.worldsibu.Manufacturer"}}
[ Manufacturer {
    _id: 'MNF_1',
    _name: 'manufacturer1',
    _productsAvailable: 2,
    _rawMaterialAvailable: 525,
    _type: 'io.worldsibu.Manufacturer' },
  Manufacturer {
    _id: 'MNF_2',
    _name: 'manufacturer2',
    _productsAvailable: 44,
    _rawMaterialAvailable: 290,
    _type: 'io.worldsibu.Manufacturer' } ]
debug: [StubHelper] Query: {"selector":{"type":"io.worldsibu.Supplier"}}
[ Supplier {
    _id: 'SPL_1',
    _name: 'supplier1',
    _rawMaterialAvailable: 2300,
    _type: 'io.worldsibu.Supplier' },
  Supplier {
    _id: 'SPL_2',
    _name: 'supplier2',
    _rawMaterialAvailable: 4850,
    _type: 'io.worldsibu.Supplier' } ]
debug: [Chaincode] ============= END : supplychainchaincode_getAllModels ===========
```
If you don't have a specific docker situation, the first image you display with the ``docker ps -a`` command is one that executes the chaincode, in that case a nice command that saves you the time to read the image id and copy/paste every time is:
```
docker logs $(docker ps -qa | head -n 1) -f
```

## Node JS backend

For the instructions on how to generate a Node JS backend you can check the project https://github.com/worldsibu/convector-rest-api where the procedure for generating the rest API is explained using this project as sample project.
