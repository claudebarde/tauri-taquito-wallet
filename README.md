# Tezos desktop wallet with Tauri and Taquito

### To run the development server:

`npm run tauri dev`

### To bundle the app:

`npm run tauri:build`

> WARNING!  
> This app was created for a tutorial meant to show how Tezos developers can use Taquito to make desktop apps.
> It isn't safe to use "as is" as there is no protection for the private key you input.  
> Please, do NOT use any personal private key, even on testnet, if you have funds on this account on mainnet!

NOTE:  
The config for Tauri was generated under MacOS. You may have to update it or reinstall Tauri for another OS. Then you can change the following fields in the `tauri.conf.json` file:

```
"build": {
    "beforeDevCommand": "npm run dev",
    "beforeBuildCommand": "npm run build",
    "devPath": "http://localhost:3030",
    "distDir": "../dist"
  }
```

and

```
"tauri": {
    ...,
    "bundle": {
        ...,
        "identifier": "tauri.taquito.wallet",
    }
}
```
