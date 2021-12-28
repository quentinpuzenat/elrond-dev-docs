## Explication du smart contract Adder

Comment stocker une variable **on-chain** sur la blockchain Elrond grâce à un smart contract? C'est ce que nous allons vous expliquer dans ce tutoriel qui va s'intéresser au code Rust d'un smart contract très simple écrit par la team Elrond sur ce [github](https://github.com/ElrondNetwork/elrond-wasm-rs/blob/v0.18.1/contracts/examples/adder/src/adder.rs), le smart contrat _Adder_.

Mais qu'est-ce que le stockage **on-chain** ?

Le stockage on-chain est le stockage de données directement sur la blockchain alors que le stockage **off-chain** est un stockage sur un serveur extérieur. Vous l'aurez compris le stockage on-chain a un coût car vous forcez alors les différents validateurs du réseau à stocker votre valeur, image, fichier...

Dans le cas des **NFT** par exemple, il est souvent préférable d'effectuer un stockage off-chain de votre image sur un système de stockage décentralisé, comme Arweave ou IPFS par exemple, et de ne stocker que l'url de votre image de façon **on-chain**.

Rentrons maintenant dans le vif du sujet!

### Le smart contract dans sa globalité

Voici le code en **Rust** du smart contract qui nous intéresse:

> Je pars du principe que vous avez des notions de **Rust** pour suivre ce tutoriel. Si ce n'est pas le cas voici une source en français: [Le Rust book en français](https://jimskapt.github.io/rust-book-fr/)

```rust
#![no_std]

elrond_wasm::imports!();

/// L'un des smarts contrats les plus simples possibles,
/// Il contient une seule variable en stockage, que n'importe qui peut incrémenter.
#[elrond_wasm::contract]
pub trait Adder {

    // intitialisation de notre smart contract
    #[init]
    fn init(&self, initial_value: BigInt) {
        self.sum().set(&initial_value);
    }

    // fonction utile au stockage et à la consultation de notre valeur
    #[view(getSum)]
    #[storage_mapper("sum")]
    fn sum(&self) -> SingleValueMapper<BigInt>;

    // Ajoutez la quantité souhaitée à la variable de stockage
    #[endpoint]
    fn add(&self, value: BigInt) -> SCResult<()> {
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

Quand à `elrond_wasm::imports!();`, on utilise la macro `imports` qui nous sert simplement à importer tout ce qui va nous être utile pour la création de smart contracts en Rust sur la blockchain Elrond.

### Création d'un smart contract

```rust
#[elrond_wasm::contract] // annotation nécessaire pour un smart contract
pub trait Adder {

    #[init] // annotation nécessaire pour la fonction d'initialisation de notre smart contract
    fn init(&self, initial_value: BigInt) {
        self.sum().set(&initial_value); // on stocke la valeur de initial_value
    }
}
```

L'annotation `#[elrond_wasm::contract]` sert à indiquer que le trait `Adder` est le trait qui contient toute la logique de notre smart contract. Il faut absolument le renseigner pour créer notre smart contract, il rend légitime toute la logique de ce dernier.

L'annotation `#[init]` sert à indiquer que la fonction qui suit cette annotation (dans notre cas la fonction `init`) ne doit être appelé qu'au déploiement de notre smart contract. Ici, on voit que notre fonction `init` prend en argument `&self` et `initial_value` et qu'elle stocke la valeur de `initial_value`. Ce stockage est possible grâce à l'appel de notre fonction `sum`.

### Stocker une variable

Pour comprendre comment on stocke la valeur voulue à la ligne 13 de notre programme, penchons nous sur la fonction de stockage `sum`.

```rust
    // fonction utile au stockage et à la consultation de notre valeur
    #[view(getSum)]
    #[storage_mapper("sum")]
    fn sum(&self) -> SingleValueMapper<BigInt>;
```

Intéressons nous à l'annotation `#[storage_mapper("sum")]`.

Cette annotation nous indique que notre fonction est en réalité un `StorageMapper` appelé `SingleValueMapper` comme on peut le voir dans ce que retourne notre fonction. Ce `SingleValueMapper` permet, en bref, de stocker une variable avec `set`et de connaître la valeur stockée avec `get`. Ainsi, lorsqu'on appelle notre fonction `sum` dans `init`:

```rust
self.sum().set(&initial_value); // on stocke la valeur de initial_value
```

On indique que nous voulons accéder à notre `SingleValueMapper` avec `self.sum()` et ensuite nous appelons `set` avec comme argument `&initial_value`. Nous venons donc de stocker la valeur `&initial_value` sur la blockchain.

De la même manière, pour connaître la valeur stockée nous avons seulement à écrire:

```rust
self.sum().get(); // on retrouve la valeur stockée
```

Pour plus de détails, et pour voir les différents `StorageMapper` possibles, voici la [documentation associée](https://docs.rs/elrond-wasm/0.22.1/elrond_wasm/storage/mappers/index.html).

La deuxième annotation `#[view(getSum)]` permet de rendre la fonction visible par des smarts contracts ou programmes extérieurs.

### Mettre à jour notre variable

Maintenant que vous savez stocker une variable et connaître sa valeur une fois stockée, il ne vous reste plus qu'à pouvoir la mettre à jour. Vous pouvez tout simplement choisir de la mettre à jour en renseignant la valeur que vous voulez dans `set` mais il parait plus intéressant d'aussi pouvoir la mettre à jour en se basant sur sa valeur déjà stockée.

Pour cela, vous avez la possibilité d'utiliser `update` qui prendra en argument une [**closure**](https://jimskapt.github.io/rust-book-fr/ch13-01-closures.html) (ou fermeture en français).

```rust
    // Ajoutez la quantité souhaitée à la variable de stockage
    #[endpoint] // pareil que pour l'annotation #[view]
    fn add(&self, value: BigInt) -> SCResult<()> {
        self.sum().update(|sum| *sum += value);

        Ok(())
    }
```

La fonction `add` répond à notre besoin avec l'utilisation de `update` et de la closure qui prend `sum` en argument (`sum` représente la valeur que l'on a déjà en stockage) et renvoie `sum` plus une valeur `value` que l'on renseigne en argument lorsqu'on appelle la fonction `add`.

Ainsi, on a ce comportement:

```rust
fn abc(&self) -> BigInt {
    let my_big_int = BigInt::from_i32(2)
    self.sum().set(&my_big_int); // on stocke 2 (ici un BigInt) qui est la valeur de my_big_int
    self.add(BigInt::from_i32(4)); // on ajoute 4 (ici un BigInt) avec la fonction add
    self.sum().get() // on renvoie 6 (ici un BigInt)
}
```

Enfin, il est important de noter que add renvoie un `SCResult<()>`. Ici, ce `SCResult<()>` est utile pour savoir si une erreur se produit durant l'appel à `add`. Pour plus d'infos sur `SCResult<()>`, allez sur cette [documentation](https://docs.rs/elrond-wasm/0.22.1/elrond_wasm/types/enum.SCResult.html).

### Conclusion

Vou savez maintenant stocker une valeur **on-chain** et vous savez utiliser le `StorageMapper` le plus simple de la documentation Elrond. Vous savez aussi créer un smart contract sur la blockchain Elrond avec une implémentation simple.
N'hésitez pas à me faire des retours et à me suivre sur twitter: [@big_q\_\_](https://twitter.com/big_q__)
