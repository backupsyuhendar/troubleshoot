# Error proton native

## Habis melakukan :
```
npx proton-native-cli init my-app

# trus masuk

npm run dev

```

## Error :
```
[webpack-cli] Error: [webpack-node-externals] : Option 'whitelist' is not supported. Did you mean 'allowlist'? 
```

## Solusi : 
- ubah `whitelist` di file config webpack menjadi `allowlist`
