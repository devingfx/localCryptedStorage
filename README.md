# localSecureStorage
localStorage with crypto

## install in DevTools
Open your devtool console and paste this:
```
localStorage.localCryptedStorage = `
let localCryptedStorage = new Proxy(localStorage,{
	
	get:function(o,p){
		// if( p == "algo" ) return {name: "AES-GCM", iv: Uint8Array.from([120,1,248,135,62,71,87,156,92,67,155,37])}
		if( p in o ){
			alert('Hello, this data is sensitive, you will be prompted for password.')
			let pass = prompt( "Enter password for "+p )
			const alg = {name: "AES-GCM", iv: Uint8Array.from([120,1,248,135,62,71,87,156,92,67,155,37])}
			return crypto.subtle.digest('SHA-256', new TextEncoder().encode(pass))
				.then( pwHash=> crypto.subtle.importKey('raw', pwHash, alg, false, ['decrypt']) )
				.then( key=> crypto.subtle.decrypt(alg, key, Uint8Array.from(o[p].split('O').map(Number)).buffer ) 	)
				.then( ptBuffer=> new TextDecoder().decode(ptBuffer) )
		}
	},
	set:function(o,p,v){
		alert('Hello, this storage is sensitive, you will be prompted for password.')
		let pass = prompt( "Enter password for this data" )
		const alg = {name: "AES-GCM", iv: Uint8Array.from([120,1,248,135,62,71,87,156,92,67,155,37])}
		return crypto.subtle.digest('SHA-256', new TextEncoder().encode(pass))
			.then( pwHash=> crypto.subtle.importKey('raw', pwHash, alg, false, ['encrypt']) )
			.then( key=> crypto.subtle.encrypt(alg, key, new TextEncoder().encode(v)) )
			.then( ctBuffer=> o[p] = new Uint8Array(ctBuffer).toString().replace(/,/g,'O') )
			// .then( ctBuffer=> console.log(ctBuffer) )
	}
})`

```
