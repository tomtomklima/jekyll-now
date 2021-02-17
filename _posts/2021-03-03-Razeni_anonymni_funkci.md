---
layout: post
title: Řazení vlastní funkcí
---

Jak řadit pomocí vlastní funkce? Přece když vestavěné řazení nestačí!


# Řazení s vlastní funkcí

Typicky se řazení s vlastní funkcí používá pro úlohy řazení objektů podle jejich vlastností, nebo při porovnávání hodnot složitějším výpočtem. Například lze řadit array objektů uživavtelů podle věku, kde máme k dispozici pouze datum narození. 

K takovémuto řazení slouží funkce `usort()` a její odvozeniny `uasort()`, která navíc zachová asociativní klíče a `uksort()`, která používá k řazení klíče. 


# Příklad s uživateli

Představme si například uživatele jako tuto třídu:

```php
<?php

class User {
	public string $name;
	public string $age;
	
	public function __construct(string $name, int $age) {
		$this->name = $name;
		$this->age = $age;
	}
}
```

Chceme všechny uživatele řadit podle věku, nejdříve ty mladší. Funkce, kterou budeme psát, příjmá reference na dva porovnávané objekty. Aby fungovala, musí vracet -1, 0 nebo 1. 

0 je potřeba vrátit, pokud jsou objekty v porovnání stejně. -1 vrátíme pro zařazení prvního objektu před druhý (posuneme ho na ose X víc do mínusu), a +1 vrátíme pro zařazení prvního objektu jako pozdějšího. Jinými slovy, pokud chceme, aby byl první objekt řazen dříve než druhý, funkce musí vrátit -1. Pokud má být řazen až za ním, funkce vrací +1. A pokud je to jedno, vrátíme 0. 

Funkci bychom mohli napsat například takto:

```php
<?php

function (User $user1, User $user2) {
	if ($user1->age < $user2->age) {
		return -1;
	}
	
	if ($user1->age > $user2->age) {
		return 1;
	}
	
	return 0;
}
```

## Spaceship operátor

Shodou náhod od PHP verze 7 přesně toto porovnání umí operátor `<=>`, pojmenovaný "spaceship operator". Funkci výše tedy můžeme zapsat mnohem pohodlněji pomocí `<=>`:

```php
<?php

function (User $user1, User $user2) {
	return $user1->age <=> $user2->age;
}
```

Pokud chceme řadit podle jména, začneme porovnávat text místo čísel. Jde použít přímo `$user1->name <=> $user2->name`, nicméně to není [binary-safe](https://stackoverflow.com/questions/3264514/in-php-what-does-it-mean-by-a-function-being-binary-safe). 

## Multibyte texty

Aby naše porovnávání fungovalo i s multibyte textem (například s českými čárkami a háčky), musíme obalit před porovnáním texty funkcí `strcmp()`. Ta umí texty porovnat bez nepředvídatelných výsledků při použití diakritiky. Pokud chceme porovnávat bez závislosti na velikosti písmen, můžeme k tomu použít `strcasecmp()`. 

Funkce řazení podle jména se zanedbáním velikosti písmen tedy může vypadat třeba:

```php
<?php

function (User $user1, User $user2) {
	return strcasecmp($user1->name, $user2->name);
}
```

# Funkční příklad

A jak tuto funkci použijeme? Jenoduše - funkci na porovnání vložíme na druhé místo parametru jako **anonymní funkci** (jsou i další způsoby, my se teď přidržíme tohoto). Funkční řazení podle věku bude vypadat takto:

```php
<?php

// definujeme, jak má vypadat uživatel, kterého budeme řadit
class User {
	public string $name;
	public string $age;
	
	public function __construct(string $name, int $age) {
		$this->name = $name;
		$this->age = $age;
	}
}

// založíme náhodné uživatele
$users = [
	new User('Jan', 28),
	new User('Josef', 7),
	new User('Jaroslav', 30),
];

// seřadíme uživatele
usort($users, function (User $user1, User $user2) {
	// naše funkce, která umí porovnávat uživatele podle věku
	return $user1->age <=> $user2->age;
});

// vypíšeme seřazené uživatele
var_dump($users);
```


# Trivia

Pozor! Když nebude mít array alespoň dvě položky, při použití řadících funkcí uksort(), uasort() a usort() se anonymní funkce vůbec **nezavolá**! Správně byste to vůbec nemuseli brát v potaz, protože porovnávací anonymní funkce by neměla mít vedlejší účinky. Je dobré na to ale pro úplnost pamatovat. 


# Too Long, Didn't Read?

Řadit array v PHP lze i pomocí vlastní řadící funkce. Nejčastěji se používá anonymní funkce vložená do volání funkce **usort()**, **uksort()** nebo **uasort()**. Řadit se dá jak podle hodnoty, tak i podle klíče. 

Ve vlastních řazeních se dá elegantně použít od PHP verze 7 [spaceship operátor `<=>`](https://www.php.net/manual/en/migration70.new-features.php#migration70.new-features.spaceship-op). 
