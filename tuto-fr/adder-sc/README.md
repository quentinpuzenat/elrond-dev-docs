## Explication du smart contract Adder

Comment stocker une variable **on-chain** sur la blockchain Elrond grâce à un smart contract? C'est ce que nous allons vous expliquer dans ce tutoriel qui va s'intéresser au code Rust d'un smart contract très simple écrit par la team Elrond sur ce [github](https://github.com/ElrondNetwork/elrond-wasm-rs/blob/v0.18.1/contracts/examples/adder/src/adder.rs), le smart contrat _Adder_.

Mais qu'est-ce que le stockage **on-chain** ?

Le stockage on-chain est le stockage de données directement sur la blockchain alors que le stockage **off-chain** est un stockage sur un serveur extérieur. Vous l'aurez compris le stockage on-chain a un coût car vous forcez alors les différents validateurs du réseau à stocker votre valeur, image, fichier...

Dans le cas des **NFT** par exemple, il est souvent préférable d'effectuer un stockage off-chain de votre image sur un système de stockage décentralisé, comme Arweave ou IPFS par exemple, et de ne stocker que l'url de votre image de façon **on-chain**.

Rentrons maintenant dans le vif du sujet!

### Le smart contract dans sa globalité

Voici le code en **Rust** du smart contract qui nous intéresse:

> Je pars du principe que vous des notions de **Rust** pour suivre ce tutoriel. Si ce n'est pas le cas voici une source en français: [Le Rust book en français](https://jimskapt.github.io/rust-book-fr/)

```rust
#![no_std]

elrond_wasm::imports!();

/// One of the simplest smart contracts possible,
/// it holds a single variable in storage, which anyone can increment.
#[elrond_wasm::derive::contract]
pub trait Adder {
    #[view(getSum)]
    #[storage_mapper("sum")]
    fn sum(&self) -> SingleValueMapper<Self::Storage, Self::BigInt>;

    #[init]
    fn init(&self, initial_value: Self::BigInt) {
        self.sum().set(&initial_value);
    }

    /// Add desired amount to the storage variable.
    #[endpoint]
    fn add(&self, value: Self::BigInt) -> SCResult<()> {
        self.sum().update(|sum| *sum += value);

        Ok(())
    }
}
```

### Mise en place de l'environnement

```rust
#![no_std]

elrond_wasm::imports!();
```

La bibliothèque standard de Rust fournit de nombreuses fonctionnalités utiles, mais prend en charge diverses fonctionnalités de son système hôte : threads, mise en réseau, allocation sur le tas (le heap pour les anglophones) et autres. Cependant, certains systèmes ne disposent pas de ces fonctionnalités, et c'est le cas de la blockchain Elrond !

Mais il n'y a pas de soucis, Rust peut également fonctionner avec ces systèmes ! Pour ce faire, nous disons à Rust que nous ne voulons pas utiliser la bibliothèque standard via un attribut : `#![no_std]`.

On veillera donc à mettre `#![no_std]` au début de nos différents codes Rust lorsqu'on codera nos smarts contracts.

Quand à `elrond_wasm::imports!();`, on utilise la macro `imports` qui nous sert simplement à importer toutes les fonctions, traits, types, macros etc.. utiles pour la création de smart contracts en Rust sur la blockchain Elrond.
