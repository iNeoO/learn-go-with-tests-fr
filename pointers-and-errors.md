# Pointeurs et erreurs

[**Vous pouvez trouver tout le code de ce chapitre ici](https://github.com/quii/learn-go-with-tests/tree/main/pointers)

Nous avons appris les structs dans la section précédente qui nous permettent de capturer un certain nombre de valeurs liées autour d'un concept.

À un moment donné, vous pourriez souhaiter utiliser des structs pour gérer l'état, en exposant des méthodes pour permettre aux utilisateurs de changer l'état d'une manière que vous pouvez contrôler.

**La fintech adore Go** et, euh, les bitcoins ? Alors montrons quel système bancaire incroyable nous pouvons créer.

Créons une struct `Portefeuille` qui nous permet de déposer des `Bitcoin`.

## Écrivez le test d'abord

```go
func TestPortefeuille(t *testing.T) {

	portefeuille := Portefeuille{}

	portefeuille.Deposer(10)

	solde := portefeuille.Solde()
	attendu := 10

	if solde != attendu {
		t.Errorf("solde %d, attendu %d", solde, attendu)
	}
}
```

Dans le [précédent exemple](https://goosegeesejeez.gitbook.io/apprendre-go-par-les-tests/fondamentaux-de-go/structs-methods-and-interfaces) nous accédions aux attributs directement par leur nom, cependant dans notre *portefeuille très sécurisé* nous ne voulons pas exposer notre état interne au reste du monde. Nous voulons en contrôler l'accès via des méthodes

## Essayez d'exécuter le test

`./portefeuille_test.go:7:9: undefined: Portefeuille`

## Écrivez la quantité minimale de code pour que le test s'exécute et vérifiez la sortie du test qui échoue

Le compilateur ne sait pas ce qu'est un `Portefeuille` alors indiquons-le-lui.

```go
type Portefeuille struct {}
```

Maintenant que nous avons déclaré notre portefeuille, essayons de relancer les tests

```
./portefeuille_test.go:9:8: portefeuille.Deposer undefined (type Portefeuille has no field or method Deposer)
./portefeuille_test.go:11:15: portefeuille.Solde undefined (type Portefeuille has no field or method Solde)
```

nous avons besoin de définir ces méthodes.

Souvenez vous de juste faire en sorte que le test se lance. nous devons être sur que notre test échoue correctement avec un message d'erreur clair.

```go
func (p Portefeuille) Deposer(montant int) {

}

func (p Portefeuille) Solde() int {
	return 0
}
```

Si le code ne semble pas familié retournew lire la section sur les structs.

le test devrait compiler et se lancer

`portefeuille_test.go:13: solde 0, attendu 10`

## Écrivez assez de code pour le faire passer

Nous avons besoin d'une variable à l'intérieur de notre struct pour stocker le solde.

```go
type Portefeuille struct {
	solde int
}
```

Dans Go si un symbol (variables, types, fonctions, et al) commencent avec une minuscule alors il est privé par rapport *à l'extérieur du package où il est définit*

dans notre cas nous voulons que notre méthode soit capable de manipuler ce solde, mais personne d'autre.

Souvenez-vous nous pouvons accéder a ce `solde` attribut dans la structure en utilisant la variable de "récupération"

```go
func (p Portefeuille) Deposer(montant int) {
	p.solde += montant
}

func (p Portefeuille) Solde() int {
	return p.solde
}
```

Avec notre carrière sécurisé dans la fintech, lancez les tests et nous reposer

`portefeuille_test.go:13: solde 0, attendu 10`

### Ce n’est pas tout à fait juste.

Eh bien, c’est déroutant : notre code semble correct. Nous ajoutons le nouveau montant au solde, et la méthode du solde devrait ensuite renvoyer son état actuel.

dans Go, **Cans on appelle une fonction ou une methode les arguments sont** ***copiées***.

Lorsqu'on appelle `func (p Portefeuille) Deposer(solde int)` the `p` est une copy de ce qui est appelé par la méthode.

Sans aller profondément, lorsque l'on crée une valeur - comme portefeuille, est elle est stocké quelque part dans la mémoire. Vous pouvez *l'adresse* de ce bout de mémoire avec `&myVariable`.

Testons en ajoutant des affichages à notre code

```go
func TestPortefeuille(t *testing.T) {

	portefeuille := Portefeuille{}

	portefeuille.Deposer(10)

	solde := portefeuille.Solde()

	fmt.Printf("address du solde dans le test est %p \n", &portefeuille.solde)

	attendu := 10

	if solde != attendu {
		t.Errorf("got %d want %d", solde, attendu)
	}
}
```

```go
func (p Portefeuille) Deposer(montant int) {
	fmt.Printf("adresse du solde dans Deposer est %p \n", &p.solde)
	p.solde += montant
}
```

L’espace réservé `%p` affiche les adresses mémoire en notation hexadécimale (base 16) avec le préfixe `0x`, et le caractère d’échappement \n imprime un saut de ligne. Remarquez que l’on obtient le pointeur (l’adresse mémoire) d’un élément en plaçant le symbole `&` devant son nom.

Maintenant relancez le test

```
adresse du solde dans Deposer est 0xc420012268
adresse odu solde dans le test est 0xc420012260
```

Vous pouvez voir que les adresses des deux soldes sont différentes. Ainsi, lorsque nous modifions la valeur du solde dans le code, nous travaillons sur une copie de celle provenant du test. Par conséquent, le solde dans le test reste inchangé.

Nous pouvons corriger cela avec des *pointeurs*. [Les pointeurs](https://gobyexample.com/pointers) nous permettent de *référencer* certaines valeurs et de les modifier. Ainsi, plutôt que de faire une copie complète du Portefeuille, nous prenons un pointeur vers celui-ci afin de pouvoir modifier les valeurs originales qu’il contient.

```go
func (p *Portefeuille) Deposer(montant int) {
	p.solde += montant
}

func (p *Portefeuille) Solde() int {
	return p.solde
}
```

La différence est que le type du récepteur est `*Portefeuille` plutôt que `Portefeuille`, ce qui peut se lire comme "un pointeur vers un portefeuille".

Essayez de relancer les tests : ils devraient maintenant passer.

Vous vous demandez peut-être pourquoi ils passent. Nous n’avons pas déréférencé le pointeur dans la fonction, comme ceci :

```go
func (p *Portefeuille) Solde() int {
	return (*p).solde
}
```

et il semble que nous ayons accédé directement à l’objet. En réalité, le code ci-dessus utilisant `(*p)` est tout à fait valide. Cependant, les concepteurs de Go ont jugé cette notation trop lourde, c’est pourquoi le langage nous permet d’écrire `p.solde` sans déréférencement explicite. Ces pointeurs vers des structures ont même un nom : *pointeurs de struct*,  et ils sont [automatiquement déréférencés](https://golang.org/ref/spec#Method_values).

Techniquement, il n’est pas nécessaire de modifier `Solde` pour utiliser un récepteur par pointeur, car faire une copie du solde ne pose aucun problème. Cependant, par convention, il est recommandé de garder les types de récepteurs de vos méthodes cohérents pour plus de constance.

## Refactoriser

Nous avons dit que nous créions un portefeuille Bitcoin, mais nous n’en avons pas encore vraiment parlé. Nous avons utilisé `int` car c’est un bon type pour compter des choses !

Créer une `struct` pour cela semblerait un peu excessif. `int` fonctionne très bien, mais il n’est pas très descriptif.

Go permet de créer de nouveaux types à partir de types existants.

La syntaxe est : `type MonNom TypeOriginal`

```go
type Bitcoin int

type Portefeuille struct {
	solde Bitcoin
}

func (p *Portefeuille) Deposer(montant Bitcoin) {
	p.solde += montant
}

func (p *Portefeuille) Solde() Bitcoin {
	return p.solde
}
```

```go
func TestPortefeuille(t *testing.T) {

	portefeuille := Portefeuille{}

	portefeuille.Deposer(Bitcoin(10))

	solde := portefeuille.Solde()

	attendu := Bitcoin(10)

	if solde != attendu {
		t.Errorf("solde %d, attendu %d", solde, attendu)
	}
}
```

Pour pouvoir utiliser `Bitcoin` avec le code du test, nous devons convertir un entier en `Bitcoin` avec la syntaxe `Bitcoin(999)`.

Un aspect intéressant de Go est que la méthode `Stringer` du package `fmt` vous permet de définir comment votre type est imprimé lorsqu'il est utilisé avec la chaîne de format `%s` (%s pour "string").

Ajoutons cette fonctionnalité [Stringer](https://golang.org/pkg/fmt/#Stringer) à `Bitcoin`:

```go
type Stringer interface {
	String() string
}
```

Cette interface est définie dans le package `fmt` et elle vous permet de définir comment votre type est affiché lorsqu’il est utilisé avec la chaîne de format `%s` lors de l’impression.

```go
func (b Bitcoin) String() string {
	return fmt.Sprintf("%d BTC", b)
}
```

Comme vous pouvez le voir, la syntaxe pour créer une méthode sur une déclaration de type est la même que pour une `struct`.

Nous devons maintenant mettre à jour les chaînes de format de nos tests afin qu’elles utilisent `String()` à la place.

```go
if solde != attendu {
	t.Errorf("solde %s attendu %s", solde, attendu)
}
```

Pour voir cela en action, cassez volontairement le test afin d’observer le résultat:

`portefeuille_test.go:18: obtenu 10 BTC, attendu 20 BTC`

Cela rend plus clair ce qui se passe dans notre test.

La prochaine fonctionnalité à implémenter est une fonction `Retirer`.

## Écrivez le test d'abord

À peu près l'inverse de `Deposer()`

```go
func TestPortefeuille(t *testing.T) {

	t.Run("Deposer", func(t *testing.T) {
		portefeuille := Portefeuille{}

		portefeuille.Deposer(Bitcoin(10))

		solde := portefeuille.solde()

		attendu := Bitcoin(10)

		if solde != attendu {
		    t.Errorf("solde %s attendu %s", solde, attendu)
		}
	})

	t.Run("Retirer", func(t *testing.T) {
		portefeuille := Portefeuille{solde: Bitcoin(20)}

		portefeuille.Retirer(Bitcoin(10))

		solde := portefeuille.solde()

		attendu := Bitcoin(10)

		if solde != attendu {
		    t.Errorf("solde %s attendu %s", solde, attendu)
		}
	})
}
```

## Essayez d'exécuter le test

`./portefeuille_test.go:18:9: portefeuille.Retirer undefined (type Portefeuille has no field or method Retirer)`

## Écrivez la quantité minimale de code pour que le test s'exécute et vérifiez la sortie du test qui échoue

```go
func (p *Portefeuille) Retirer(montant Bitcoin) {

}
```

`portefeuille_test.go:33: solde 20 BTC, attendu 10 BTC`

## Écrivez assez de code pour le faire passer

```go
func (p *Portefeuille) Retirer(montant Bitcoin) {
	p.solde -= montant
}
```

## Refactoriser

Il y a des doublons dans nos tests, Refactorisons ça.

```go
func TestPortefeuille(t *testing.T) {

	verifieSolde := func(t testing.TB, portefeuille Portefeuille, attendu Bitcoin) {
		t.Helper()
		solde := portefeuille.Balance()

		if solde != attendu {
			t.Errorf("solde %s attendu %s", solde, attendu)
		}
	}

	t.Run("deposer", func(t *testing.T) {
		portefeuille := Portefeuille{}
		portefeuille.Deposer(Bitcoin(10))
		verifieSolde(t, portefeuille, Bitcoin(10))
	})

	t.Run("retirer", func(t *testing.T) {
		portefeuille := Portefeuille{solde: Bitcoin(20)}
		portefeuille.Retirer(Bitcoin(10))
		verifieSolde(t, portefeuille, Bitcoin(10))
	})

}
```

Que se passe-t-il si un utilisateur essaie de retirer plus de bitcoins qu'il n'en a ? Il finira par avoir un solde négatif.

Comment signale-t-on un problème lorsqu'on utiliser `Retirer`?

En Go, si vous souhaitez signaler une erreur, il est idiomatique que votre fonction retourne une valeur `err` que l’appelant pourra vérifier et traiter.

Éssayons ça dans le test.

## Écrivew le test en premier

```go
t.Run("Retirer avec des fonds insuffisants", func(t *testing.T) {
	soldeInitial := Bitcoin(20)
	portefeuille := Portefeuille{soldeInitial}
	erreur := portefeuille.Retirer(Bitcoin(100))

	verifieSolde(t, portefeuille, soldeInitial)

	if erreur == nil {
		t.Errorf("une erreur aurait dû être retournée")
	}
})
```

Nous voulons que `Retirer` retourne une erreur *si* vous essayez de retirer plus d’argent que vous n’en avez, et que le solde reste inchangé.

Nous vérifions ensuite qu’une erreur a bien été retournée, en échouant le test si sa valeur est `nil`.

`nil` est l’équivalent de `null` dans d’autres langages de programmation. Les erreurs peuvent être `nil`, car le type de retour de `Retirer` sera `error`, qui est une interface. Si vous voyez une fonction qui prend ou retourne des interfaces, ces valeurs peuvent être nulles (`nillable`).

Comme pour `null`, si vous essayez d’accéder à une valeur `nil`, le programme lèvera une **panique à l’exécution (runtime panic)**. C’est problématique ! Vous devez donc toujours vérifier les valeurs `nil`.

## Essayez d'exécuter le test

`./portefeuille_test.go:37:15: portefeuille.Retirer(Bitcoin(100)) used as value`

La formulation est peut-être un peu ambiguë, mais notre intention précédente avec `Retirer` était simplement de l’appeler, sans qu’elle ne retourne de valeur. Pour que le code compile, nous devons la modifier afin qu’elle ait un type de retour.

## Écrivez la quantité minimale de code pour que le test s'exécute et vérifiez la sortie du test qui échoue

```go
func (p *Portefeuille) Retirer(montant Bitcoin) error {
	p.solde -= montant
	return nil
}
```

Encore une fois, il est très important d’écrire uniquement le code nécessaire pour satisfaire le compilateur. Nous corrigeons donc notre méthode `Retirer` afin qu’elle retourne une valeur de type `error`. Et pour l’instant, comme il faut bien retourner *quelque chose*, retournons simplement `nil`.

## Écrivez assez de code pour le faire passer

```go
func (p *Portefeuille) Retirer(montant Bitcoin) error {

	if montant > p.solde {
		return errors.New("oh non")
	}

	p.solde -= montant
	return nil
}
```

N'oubliew pas d'import `errors` dans votre code.

`errors.New` Crée une nouvelle `error` avec un message de votre choix.

## Refactoriser

Créons rapidement une fonction d’aide pour la vérification des erreurs, afin d’améliorer la lisibilité de notre test.

```go
verifieErreur := func(t Testing.TB,  erreur error, attendu error) {
	t.Helper()
	if erreur != nil {
		t.Error("une erreur était attendue, mais aucune n’a été retournée")
	}
}
```

Et dans notre test

```go
t.Run("Retirer avec des fonds insuffisants", func(t *testing.T) {
	soldeInitial := Bitcoin(20)
	portefeuille := Portefeuille{soldeInitial}
	erreur := portefeuille.Retirer(Bitcoin(100))

    verifieErreur(t, erreur)
	verifieSolde(t, portefeuille, soldeInitial)
})
```

Espérons qu’en retournant une erreur comme "oh non", vous pensiez déjà que nous allions probablement itérer dessus,  car ce n’est pas très utile de renvoyer un tel message.

En supposant que l’erreur soit finalement retournée à l’utilisateur, mettons à jour notre test afin de vérifier le contenu du message d’erreur, et non pas seulement la présence d’une erreur.

## Écrivons le test en premier

Mettons à jour notre fonction d’aide pour qu’elle compare les erreurs à une valeur de type `string`.

```go
verifieErreur := func(t Testing.TB,  erreur error, attendu string) {
	t.Helper()

    if attendu == nil {
        t.Fatal("aucune erreur reçue, mais une était attendue")
    }

	if attendu.Error() != attendu {
	    t.Errorf("erreur reçue '%v', erreur attendue '%v'", erreur, attendu)
	}
}
```

Comme vous pouvez le voir, les valeurs de type `error` peuvent être converties en chaîne de caractères grâce à la méthode `.Error()`, que nous utilisons ici pour la comparer à la chaîne attendue. Nous vérifions également que l’erreur n’est pas `nil`, afin d’éviter d’appeler `.Error()` sur une valeur nulle.

Puis mettez à jour l’appelant en conséquence.

```go
t.Run("Retirer avec des fonds insuffisants", func(t *testing.T) {
	soldeInitial := Bitcoin(20)
	portefeuille := Portefeuille{soldeInitial}
	erreur := portefeuille.Retirer(Bitcoin(100))

	assertError(t, erreur, "retrait impossible, solde insuffisant")
	assertBalance(t, portefeuille, soldeInitial)
})
```

Nous avons introduit `t.Fatal`, qui arrête immédiatement le test lorsqu’il est appelé. Cela permet d’éviter de poursuivre les vérifications sur une erreur inexistante. Sans cela, le test continuerait à l’étape suivante et provoquerait une panique due à un pointeur `nil`.

## Éssayez de lancer le test

`portefeuille_test.go:61: erreur reçue 'oh non', erreur attendue 'retrait impossible, solde insuffisant'`

## Ércrivez assez de code pour faire passer le test

```go
func (p *Portefeuille) Retirer(montant Bitcoin) error {

	if montant > p.solde {
		return errors.New("retrait impossible, solde insuffisant")
	}

	p.solde -= montant
	return nil
}
```

## Refactoriser

Nous avons une duplication du message d’erreur à la fois dans le code du test et dans le code de `Retirer`.

Ce serait vraiment agaçant que le test échoue simplement parce que quelqu’un a reformulé le message d’erreur; c’est trop de détails pour notre test. Ce qui nous importe réellement, ce n’est pas la formulation exacte du message, mais qu’une erreur significative liée au retrait soit bien retournée dans certaines conditions.

En Go, les erreurs sont des valeurs;  nous pouvons donc les refactoriser dans une variable afin d’avoir une unique source de vérité.

```go
var ErrFondsInsuffisants = errors.New("impossible de retirer: fonds insuffisants")

func (p *Portefeuille) Retirer(montant Bitcoin) error {

	if montant > p.solde {
		return ErrFondsInsuffisants
	}

	p.solde -= montant
	return nil
}
```

Le mot-clé `var` nous permet de définir des valeurs globales au package.

C’est déjà une amélioration en soi, car notre fonction `Retirer` est désormais beaucoup plus claire.

Ensuite, nous pouvons refactoriser notre code de test pour utiliser cette variable au lieu de chaînes de caractères spécifiques.

```go
func TestPortefeuille(t *testing.T) {

	t.Run("Deposer", func(t *testing.T) {
		portefeuille := Portefeuille{}
		portefeuille.Deposer(Bitcoin(10))
		verifieSolde(t, portefeuille, Bitcoin(10))
	})

	t.Run("Retirer avec des fonds", func(t *testing.T) {
		portefeuille := Portefeuille{Bitcoin(20)}
		portefeuille.Retirer(Bitcoin(10))
		verifieSolde(t, portefeuille, Bitcoin(10))
	})

	t.Run("Retirer avec des fonds insuffisants", func(t *testing.T) {
    	portefeuille := Portefeuille{Bitcoin(20)}
    	erreur := Portefeuille.Retirer(Bitcoin(100))

		verifieErreur(t, erreur, ErrFondsInsuffisants)
		verifieSolde(t, portefeuille, Bitcoin(20))
	})
}

verifieSolde := func(t testing.TB, portefeuille Portefeuille, attendu Bitcoin) {
	t.Helper()
	solde := portefeuille.Balance()

	if solde != attendu {
		t.Errorf("solde %s attendu %s", solde, attendu)
	}
}

verifieErreur := func(t Testing.TB,  erreur error, attendu string) {
	t.Helper()
    if attendu == nil {
        t.Fatal("aucune erreur reçue, mais une était attendue")
    }

	if attendu.Error() != attendu {
	    t.Errorf("erreur reçue '%v', erreur attendue '%v'", erreur, attendu)
	}
}
```

Et maintenant, le test est lui aussi plus facile à lire.

J’ai déplacé les fonctions d’aide en dehors de la fonction de test principale, afin que, lorsqu’une personne ouvre le fichier, elle puisse d’abord lire nos assertions plutôt que les fonctions utilitaires.

Une autre propriété intéressante des tests est qu’ils nous aident à comprendre l’usage *réel* de notre code, afin d’écrire un code plus intuitif. On peut voir ici qu’un développeur peut simplement appeler notre code, faire une comparaison avec `ErrFondsInsuffisants` et agir en conséquence. Maintenant nous vérifions que l'erreur a la bonne valeur, ce qui est plus précis et rend notre code plus compréhensible.

### Erreurs non vérifiées

Bien que le compilateur Go vous aide énormément, il y a parfois des choses qui peuvent encore vous échapper, et la gestion des erreurs peut parfois être délicate.

Il existe un scénario que nous n’avons pas encore testé. Pour le découvrir, exécutez la commande suivante dans un terminal afin d’installer `errcheck`, l’un des nombreux *linters* disponibles pour Go.

`go install github.com/kisielk/errcheck@latest`

Ensuite, dans le répertoire contenant votre code, exécutez la commande : `errcheck .`

Vous devriez obtenir un résultat semblable à :

`portefeuille_test.go:17:18: portefeuille.Retirer(Bitcoin(10))`

Ce message nous indique que nous n’avons pas vérifié l’erreur retournée à cette ligne de code. Sur mon ordinateur, cette ligne correspond au scénario normal de retrait, car nous n’avons pas vérifié que, lorsque `Retirer` réussit, aucune erreur n’est retournée.

Voici le code final du test qui prend cela en compte.

```go
func TestPortefeuille(t *testing.T) {

	t.Run("Deposer", func(t *testing.T) {
		portefeuille := Portefeuille{}
		portefeuille.Deposer(Bitcoin(10))

		verifieSolde(t, portefeuille, Bitcoin(10))
	})

	t.Run("Retirer avec des fonds", func(t *testing.T) {
		portefeuille := Portefeuille{Bitcoin(20)}
		erreur = portefeuille.Retirer(Bitcoin(10))

		verifieNoErreur(t, err)
		verifieSolde(t, portefeuille, Bitcoin(10))
	})

	t.Run("Retirer avec des fonds insuffisants", func(t *testing.T) {
    	portefeuille := Portefeuille{Bitcoin(20)}
    	erreur := Portefeuille.Retirer(Bitcoin(100))

		verifieErreur(t, erreur, ErrFondsInsuffisants)
		verifieSolde(t, portefeuille, Bitcoin(20))
	})
}

verifieSolde := func(t testing.TB, portefeuille Portefeuille, attendu Bitcoin) {
	t.Helper()
	solde := portefeuille.Balance()

	if solde != attendu {
		t.Errorf("solde %s attendu %s", solde, attendu)
	}
}

func verifieNoErreur(t testing.TB, attendu error) {
    t.Helper()
    if attendu != nil {
        t.Fatal("une erreur a été retournée alors qu’aucune n’était attendue")
    }
}

verifieErreur := func(t Testing.TB,  erreur error, attendu string) {
	t.Helper()
    if attendu == nil {
        t.Fatal("aucune erreur reçue, mais une était attendue")
    }

	if attendu.Error() != attendu {
	    t.Errorf("erreur reçue '%v', erreur attendue '%v'", erreur, attendu)
	}
}
```

## En résumé

### Pointeurs

* Go copie les valeurs quand vous les passez aux fonctions/méthodes, donc si vous écrivez une fonction qui a besoin de muter l'état vous aurez besoin qu'elle prenne un pointeur vers la chose que vous voulez changer.
* Le fait que Go prenne une copie des valeurs est utile beaucoup de fois mais parfois vous ne voudrez pas que votre système fasse une copie de quelque chose, auquel cas vous avez besoin de passer une référence. Les exemples incluent référencer de très grandes structures de données ou des choses où seule une instance est nécessaire \(comme les pools de connexion de base de données\).

### nil

* Les pointeurs peuvent être nil
* Quand une fonction retourne un pointeur vers quelque chose, vous devez vous assurer de vérifier s'il est nil ou vous pourriez lever une exception d'exécution - le compilateur ne vous aidera pas ici.
* Utile quand vous voulez décrire une valeur qui pourrait manquer

### Erreurs

* Les erreurs sont la façon de signifier l'échec quand on appelle une fonction/méthode.
* En écoutant nos tests nous avons conclu que vérifier une chaîne dans une erreur résulterait en un test fragile. Donc nous avons refactorisé notre implémentation pour utiliser une valeur significative à la place et cela a résulté en du code plus facile à tester et nous avons conclu que ce serait plus facile pour les utilisateurs de notre API aussi.
* Ce n'est pas la fin de l'histoire avec la gestion d'erreurs, vous pouvez faire des choses plus sophistiquées mais c'est juste une introduction. Les sections ultérieures couvriront plus de stratégies.
* [Ne vérifiez pas juste les erreurs, gérez-les gracieusement](https://dave.cheney.net/2016/04/27/dont-just-check-errors-handle-them-gracefully)

### Créer de nouveaux types à partir de types existants

* Utile pour ajouter plus de signification spécifique au domaine aux valeurs
* Peut vous permettre d'implémenter des interfaces

Les pointeurs et les erreurs sont une grande partie de l'écriture de Go avec laquelle vous devez être à l'aise. Heureusement le compilateur vous aidera _habituellement_ si vous faites quelque chose de mal, prenez juste votre temps et lisez l'erreur.
