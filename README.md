## Vue js Authentication 


### Installation

```bash
npm install
```
> Note: You need to have nodejs installed on your machine

### Run

```bash
npm run serve
```

> Note: If you want to run the project on a different port, you can do so by running the following command

```bash
npm run serve -- --port 8081
```

> Note : If you got an error like this

```bash
Error - "digital envelope routines::unsupported ... ERR_OSSL_EVP_UNSUPPORTED"\
```
- change the package.json file 
```json
// from this

"scripts": {
    "serve": "vue-cli-service serve",
    "build": "vue-cli-service build",
    "lint": "vue-cli-service lint"
},
```
```json
// to this

"scripts": {
    "serve": "export NODE_OPTIONS=--openssl-legacy-provider && vue-cli-service serve",
    "build": "export NODE_OPTIONS=--openssl-legacy-provider && vue-cli-service build",
    "lint": "export NODE_OPTIONS=--openssl-legacy-provider && vue-cli-service lint"
},
```

### Dependencies used
- Axios
    - For sending and retrieving data from our API
- Vuex
    - For storing data gotten from our API
- Vue-Router
    - For navigation and protection of Routes

### Project Setup

- Look at setup.md file