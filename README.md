# E2EE Todo App

This is a simple todo web app, inspired by TodoMVC, and supported by SvelteKit.
It has basic CRUD abilites, meaning you can create, view, update and delete your todos.

### Application Architecture

The rough application architecture can be seen on [this](https://drive.google.com/file/d/1PMfS435HpT5ICBRcgiRqN9r_kv8vFhAE/view?usp=sharing) collection of diagrams.
The most demanding part here was solving all the authentication/cryptography on the browser side, like generating the public key from the private key, generating a key pair as a whole etc.
For this, the frontend utilizes [forge](https://github.com/digitalbazaar/forge), a native implementation of many cryptographic tools.

## Getting started

To get started with this project, follow these steps:

1. Clone the repository to your local machine

```bash
git clone https://github.com/flawnn/E2EETodo.git
```

2. Install the dependencies

```bash
npm install
```

3. Start the development server with hot reload

```bash
npm run dev
```

This will start the development server with hot reload enabled. You can now make changes to the code and see the changes immediately in the browser.

Optionally you can run `npm run build` to generate needed files for a production deployment.

## License

This project is licensed under the [MIT License](https://github.com/flawnn/E2EETodo/blob/master/LICENSE.txt).
