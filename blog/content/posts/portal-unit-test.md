---
title: "Portal Unit Test"
date: 2022-02-21T13:52:20+01:00
draft: false
---

# Table des matières

[1 Organisation des tests unitaires (back-end)](#§1-organisation-des-tests-unitaires)

[2 Classes de tests](#§2-classes-de-tests)

[3 Tests unitaires](#§3-tests-unitaires)

[4 Données de test et Fixtures](#§4-données-de-test-et-fixtures)

[5 Dépendances et Mocks](#§5-dépendances-et-mocks)

[6 Exécution des tests et Couverture de code TODO](#§6-exécution-des-tests-et-couverture-de-code)

[7 TDD et Live testing TODO](#§7-tdd-et-live-testing)

[8 Liens utiles](#§8-liens-utiles)

# §1 Organisation des tests unitaires (back-end)

Les Tests Unitaires (TU) côté back sont organisés par classe de tests, chaque classe regroupant les TU pour un (et un seul) service métier `ApplicationCore` à tester. La convention de nommage d'une classe de tests est donc simplement `<NomService>Tests`.
Par exemple, `ProductServiceTests` regroupe tous les TU pour le service métier `ProductService`, et uniquement ceux-là.

Ces classes de tests sont systématiquement localisées dans le projet `UnitTests` de chaque solution récente de Portal. Au sein de ce projet, ces classes sont organisées selon une hiérarchie de dossiers similaire à celle des services métiers testés.
Par exemple, le dossier `UnitTests/Portal_CPA.ApplicationCore/ApplicationServices` regroupe les TU liés à tous les services situés dans `ApplicationCore/ApplicationServices`.

[^retour en haut](#table-des-matières)

# §2 Classes de tests

La classe de tests est une classe C# qui regroupe l'ensemble des tests nécessaires à la couverture d'un seul et même service métier, appelé **System Under Test** (SUT). Une bonne pratique est de regrouper les tests unitaires concernant une même méthode du SUT à l'aide de régions.

Par exemple :

```csharp
#region GetClients() Tests

public GetClientsReturnsListWhen()...
public GetClientsThrowsWhen()...

#endregion
```

## §2.1 Préparation du contexte (Test Setup)

Si une partie du contexte (jeux de données ou services) est commune à tous les TU d'une classe de tests, il est possible de l'instancier une seule et même fois au niveau du **constructeur** de la classe de tests.
À noter que le constructeur sera appelé à l'exécution de _chacun_ des tests. En effet, afin d'éviter des problèmes de thread-safety (accès concurrents, deadlocks), les tests sont toujours censés pouvoir fonctionner en vase clos et être exécutés en parallèle.

Par exemple, voici comment injecter un mock de `ILoggerApp` pour chaque TU de `ServiceTests` :

```csharp
public class ServiceTests
{
    public ServiceTests()
    {
        // Setup: inject instance of mock logger for DI
        _fixture.Inject(new Mock<ILoggerApp<ServiceTests>>().Object);
    }
}
```

Pour éviter d'avoir à dupliquer le code de mise en place du contexte dans chaque classe de tests, un comportement par défaut a été défini au sein de la classe abstraite `SystemUnderTestBase` (ou `ServiceUnderTestBase` dans une ancienne version à refactoriser), qui prend en paramètres de généricité l'interface et le type du SUT.

Par exemple, pour un service métier de type Service :

```csharp
public class ServiceTests : SystemUnderTestBase<IService, Service>
{
    ...
}

// ou anciennement : (attention à l'ordre des types de généricité qui a été inversé)
public class ServiceTests : ServiceUnderTestBase<Service, IService>
{
    ...
}
```

Ce constructeur crée une fixture qui :

- fournit une injection de dépendance pour le service à tester,
- évite la création récursive d'[objets composites](https://fr.wikipedia.org/wiki/Objet_composite) indéfiniment (profondeur de récursivité limitée à 1),
- génère des mocks automatiquement lorsqu'aucun mock spécifique n'a été prévu dans le test ([AutoMock](#§5.4-injection-de-dépendances-et-automocking)).

## §2.2 Libération du contexte (Test Teardown)

De même, il est possible de libérer les ressources susceptibles de générer des fuites de mémoire (flux de données, injections de dépendance) à l'aide de l'interface IDisposable et de la méthode Dispose(). Pour plus d'informations, consulter [https://xunit.net/docs/shared-context](https://xunit.net/docs/shared-context).

[^retour en haut](#table-des-matières)

# §3 Tests unitaires

Un test unitaire se présente sous la forme d'une méthode de la classe de tests. Cette méthode de test met en place le cas à tester, et évalue son résultat. Pour ce faire, au sein d'une même méthode, il est nécessaire :

- de générer ou récupérer les données de test,
- d'instancier le SUT et ses dépendances,
- d'appeler la méthode à tester et éventuellement récupérer son résultat,
- et de vérifier que l'appel à la méthode ou la valeur de retour est conforme à ce qui était attendu.

Il est à noter que dans un TU on teste toujours **une** méthode **publique** du SUT. Les méthodes privées seront couvertes indirectement par les TU des méthodes publiques y faisant appel. Si des méthodes privées ne sont pas totalement couvertes alors que toutes les méthodes publiques ont été testées à 100%, alors on a affaire à du code mort.

## §3.1 Nommage

Une méthode de test bien nommée doit permettre de retrouver en un instant le cas testé. Pour ce faire, la convention de nommage suivante peut être utilisée : **`<NomMéthode><ActionAttendue><CasTesté>`**.

Par exemple, pour une méthode GetProduct() qui devrait renvoyer le produit demandé si l'identifiant du produit est valide, ceci donne :
`public void GetProductReturnsRequestedProductWhenIdIsValid() { ... }`

Ou pour une méthode asynchrone GetProductAsync() :
`public async Task GetProductAsyncReturnsRequestedProductWhenIdIsValid() { ... }`

Pour un cas d'erreur, on peut avoir ceci :
`public void GetProductThrowsBusinessExceptionWhenIdIsUnknown() {...}`

Cette convention de nommage pourra être améliorée si nécessaire. Pour plus d'informations : [https://stackoverflow.com/questions/155436/unit-test-naming-best-practices](https://stackoverflow.com/questions/155436/unit-test-naming-best-practices).

## §3.2 Attributs

Afin que les méthodes de tests puissent être découvertes par le lanceur de tests (Test Runner) on ajoute un attribut C# devant la déclaration de la méthode. Le framework de TU dans Portal étant `xUnit`, les attributs à utiliser sont **`[Fact]`** ou **`[Theory]`**.

Il faut utiliser l'attribut `[Fact]` lorsque le cas à tester ne nécessite pas plusieurs jeux de données. Dans le cas contraire, l'attribut `[Theory]` permet de renseigner plusieurs jeux de données qui devront impérativement être passés en paramètre de la méthode de test à l'aide d'attributs complémentaires (`[InlineData]`, `[MemberData]` ou `[ClassData]`).

Par exemple :

```csharp
// Test simple
[Fact]
public void GetProductAsyncReturnsRequestedProductWhenIdIsValid() { ... }

// Test paramétré
[Theory]
[InlineData("00.01.02.03.04", "0001020304")]
[InlineData("00-01-02-03-04", "0001020304")]
public void GetTiersReturnsCleanNumberWhenCharsSuperfluous(string inputNumber, string expectedNumber)
```

Plus d'informations sur les [tests paramétrés au §4.2](#§4.2-tests-paramétrés).

## §3.3 Arrange, Act, Assert (AAA)

Un TU se déroule toujours en trois étapes :

1. L'étape **Arrange** où on met en place le cas de test : génération ou récupération du jeu de données, instanciation du SUT et des dépendances simulées (mocks d'API, DA...) nécessaires pour la méthode testée.
2. L'étape **Act** où on appelle la méthode testée (il ne doit donc y avoir qu'une seule instruction dans cette étape normalement) et où on récupère la valeur de retour éventuelle.
3. L'étape **Assert** où on vérifie que la valeur de retour ou les données du test correspondent à ce qui était attendu. Il est également possible de vérifier qu'une méthode d'une des dépendances mockées a bien été appelée. C'est notamment utile quand on teste des méthodes qui ne retournent aucune valeur (en général les Create, Update, Delete).

Par exemple :

```csharp
[Theory]
[InlineData("9854123", "99999999", ServiceContext.OnlyQuadra, "3", 0)]
public async Task CreateTiersSmbShouldReturnIdWhenQuadraContext(string sic, string oldSic, ServiceContext contexte, string sousClientId, long tiersId)
{
    // Arrange
    // a) input data
    var tiers = new TiersProxy { TiersId = tiersId };

    // b) dependencies
    _fixture.Inject(CommonDataApiClientMocker.CreateMockWith(sic, oldSic, sousClientId, tiers));
    _fixture.Inject(ClaimsContextMocker.CreateMockWith(oldSic, sousClientId, string.Empty));

    // c) sut
    var sut = _fixture.Create<ICommonDataService>();

    // Act
    var tiersIdReturned = await sut.CreateTiersSmb(new Entreprise(), sic, string.Empty, string.Empty, contexte);

    // Assert
    tiersIdReturned.ShouldBe(tiers.TiersId);
}
```

## §3.4 Assertions

Les assertions permettent de vérifier si le résultat correspond bien à ce qui était attendu dans le test. Si une assertion est fausse, une exception est levée et le test passe au rouge, avec un message d'erreur permettant d'analyser le problème. De même, si une exception est levée à n'importe quel autre endroit du test (instanciation des services, appel de la méthode testée), il passera également au rouge.

Au contraire, si toutes les assertions sont OK, le test passe au vert. Évidemment si aucune assertion n'est faite dans le test, le test passera également au vert, donnant le sentiment que tout s'est bien passé... À éviter donc : SonarCloud émettra un warning.

Dans Portal, on utilise la bibliothèque `Shouldly` car elle permet d'écrire les assertions de manière plus lisible que les `Assert()` de .NET.

Exemples d'assertions :

```csharp
boolResult.ShouldBeTrue();                 // test de la valeur d'un booléen
productResult.NAME.ShouldBe(expectedName); // test de la valeur d'une propriété
intResult.ShouldBeInRange(1, 100);         // test d'intervalle d'un int

productResult.EMPTYFIELD.ShouldBeNull();   // test de nullité
productResult.ID.ShouldNotBeNull();        // test de non-nullité

productResult.ShouldBeOfType<PRODUCT>();   // test du type d'un objet

productsResult.ShouldAllBe(p => p.ID > 0); // test d'une condition pour tous les éléments d'une liste
productsResult.ShouldContain(product);     // test d'appartenance d'un élément à une liste

sut.AddNewUser(null)
   .ShouldThrow<Exception>();              // test d'exception pour un cas d'erreur
```

Pour plus d'informations : [documentation Shouldly](http://docs.shouldly-lib.net/v2.4.0/docs).

[^retour en haut](#table-des-matières)

# §4 Données de test et Fixtures

Les données de test peuvent être créées manuellement ou générées à l'aide de **[fixtures](https://fr.wikipedia.org/wiki/Test_fixture)**. Dans Portal, la bibliothèque utilisée pour les fixtures est `AutoFixture`.

## §4.1 Génération à l'aide d'une fixture

Pour générer des données de test (en entrée ou attendues), le plus simple est d'utiliser la fixture créée par le constructeur de la classe `SystemUnderTestBase` vue plus haut. Pour générer un objet et toutes ses propriétés aléatoirement, on procède comme suit :
`var expectedResult = _fixture.Create<ProduitInterlocuteur>();`

Voici le genre d'objet retourné :
![fixture.png](/.attachments/fixture-c5904841-4e73-4843-bdb6-8e2395f2bfd2.png)

Comme on peut le voir, les valeurs générées sont complètement aléatoires. Il se peut que certaines ne respectent pas le format souhaité, notamment si un champ de type string doit contenir une date ou un code spécifique.

Il est possible d'éviter ce problème en spécifiant des valeurs pour certains champs et en laissant les autres se générer automatiquement. Pour ce faire, on appelle d'abord la méthode `Build()` et on force la valeur des champs à l'aide d'autant d'appels à `With()` que nécessaire, puis on termine avec `Create()` :

```csharp
_fixture.Build<PaymentMethod>()
        .With(pm => pm.ExpirationMonth, "10")
        .With(pm => pm.ExpirationYear, "20")
        .Create(); // ne pas oublier le Create() final
```

On peut également faire appel plusieurs fois à la même fixture pour spécifier des objets imbriqués :

```csharp
_fixture.Build<PaymentMethod>()
        .With(pm => pm.CardHolderInfo,
              _fixture.Build<CardHolderInfo>()
                      .With(ch => ch.Email, "user@bank.com")
                      .Create())
        .Create();
```

Enfin, il est possible de générer une collection d'objets de type `IEnumerable` en appelant `CreateMany()`. Par exemple :

```csharp
var inputProducts = _fixture.CreateMany<ProduitInterlocuteur>(); // génère plusieurs éléments (nombre non précisé)

var inputProducts = _fixture.CreateMany<ProduitInterlocuteur>(5); // génère 5 éléments
```

À savoir que si un objet à générer contient une propriété de type liste, le nombre d'éléments de cette liste n'est pas défini (cas du premier `CreateMany()` ci-dessus). Il suffit cependant de spécifier leur nombre en imbriquant un second appel à `CreateMany()` (pour la propriété Contacts dans l'exemple suivant) :

```csharp
var inputCartes =
    _fixture.Build<Carte>()
            .With(c => c.Contacts,
                  fixture.CreateMany<Contact>(5).ToList()) // ToList() car le champ Contacts attend
                                                           // une liste, et non un IEnumerable
            .CreateMany()
```

Pour les objets complexes comportant de nombreuses propriétés de navigation, qui elles-mêmes mènent à des objets comportant des propriétés de navigation (entités Entity Framework notamment), il se peut que leur génération par AutoFixture prennent **plusieurs secondes voire dizaines de seconde**.

Pour contourner ce problème, on peut :

- Appeler la méthode Without() sur la ou les propriétés de navigation que l'on souhaite ignorer, afin d'améliorer quelque peu les performances de génération :

```csharp
var listCountry = _fixture.Build<Country>()
                          .Without(c => c.Address) // Ignore le champ Address, lui même lié à PersonAddress, lié à Person, etc.
                          .CreateMany(4).ToList();
```

- Configurer AutoFixture pour ignorer toutes les propriétés `virtual` des objets générés, qui correspondent habituellement aux propriétés de navigation d'Entity Framework. C'est ce qui a été fait dans la classe `SystemUnderTestBase` de PortalApi. Cela permet de réduire drastiquement la quantité d'objets imbriqués générés. En contrepartie, il faut déclarer explicitement tous les objets imbriqués nécessaires pour le TU s'ils sont contenus dans une propriété `virtual` :

```csharp
var listCountry = _fixture.Build<Country>()
                          .With(c => c.Address) // Force la génération de la propriété de navigation Address, qui est virtual.
                                                // Si Address comporte elle-même des propriétés virtual, elles seront ignorées par AutoFixture,
                                                // à moins de rajouter des With() imbriqués pour ces propriétés aussi.
                          .CreateMany(4).ToList();
```

- Créer l'objet manuellement (à éviter).

## §4.2 Tests paramétrés

Les données de test peuvent être définies au sein même du TU, dans la partie Arrange. Cependant, il peut être préférable de spécifier un ou plusieurs jeux de données en attribut de la méthode de test.
C'est notamment utile si plusieurs cas semblables peuvent être testés à l'aide de la même méthode de test. Par exemple ici, plusieurs valeurs peuvent être testées facilement au sein du même TU :

```csharp
[Theory]
[InlineData("00.01.02.03.04", "0001020304")]
[InlineData("+33(0)1-02-03-04", "331020304")]
public void GetTiersReturnsCleanNumberWhenCharsSuperfluous(string inputNumber, string expectedNumber)
{ ... }
```

On peut ainsi spécifier autant de cas de tests que nécessaires à l'aide de l'attribut `[InlineData]` en faisant simplement correspondre les différentes valeurs du jeu de données avec l'ordre et le type des paramètres attendus par la méthode.

Il est également possible de faire varier le nombre de données de test en mettant à profit le mot-clé params, ici utilisé avec le paramètre productCodes :

```csharp
[Theory]
[InlineData(ServiceContext.SMBAndCegid, "XX")]
[InlineData(ServiceContext.SMBAndQuadra, "DF", "XX")]
public void UpdateUserWorksWhenUserValid(ServiceContext serviceContext, params string[] productCodes)
{
    // Assert
    var userRequest = _fixture.Create<RequestUserServices>();
    userRequest.ServicesEtat = productCodes.Any() ? productCodes.Select(pc => new ServiceEtat { Code = pc }).ToList() : null;
    // ...
}
```

L'attribut `[InlineData]` permet donc de couvrir plus de code à l'aide d'un même TU, ce qui permet de factoriser la logique de test. **Attention à ne pas aller trop loin** cependant, en voulant gérer tous les cas possibles dans un seul et unique test (données trop différentes, ou cas nominaux et limites mélangés). Pour que le TU reste lisible et maintenable, il faut éviter d'ajouter de la complexité, en **évitant notamment de conditionner les assertions avec un if, ou pire, un switch**.

Cependant, la limite de cet attribut est atteinte lorsqu'on souhaite passer autre chose que des constantes en entrée du test, ou lorsque que l'on veut mettre en commun des jeux de données pour plusieurs TU. Dans ce cas, il sera intéressant de générer ces jeux de données à l'aide de méthodes locales à la classe de tests, ou à l'aide de classes dédiées pouvant être réutilisées par plusieurs classes de tests.

Pour ce faire, xUnit propose les attributs `[MemberData]` et `[ClassData]` : le premier permet de définir la méthode ou propriété à appeler au sein de la classe de tests, le second indique la classe dédiée à utiliser pour générer les données.

Exemples :

- Pour l'attribut `[MemberData]`, il faut indiquer le nom de la méthode génératrice de données. Cette méthode doit être statique et locale à la classe de tests, et retourner un `IEnumerable<object[]>` :

```csharp
public class CalculatorTests
{
    [Theory]
    [MemberData(nameof(CreateData))]
    public void AddReturnsExpectedValueWhenArgsValid(int value1, int value2, int expected)
    {
        var calculator = new Calculator();

        var result = calculator.Add(value1, value2);

        Assert.Equal(expected, result);
    }

    public static IEnumerable<object[]> CreateData =>
        new List<object[]>
        {
            new object[] { 1, 2, 3 },
            new object[] { -4, -6, -10 },
            new object[] { -2, 2, 0 },
            new object[] { int.MinValue, -1, int.MaxValue },
        };
}
```

- Pour l'attribut `[ClassData]`, il faut indiquer le type de la classe générant les données. Cette classe doit implémenter obligatoirement `IEnumerable<object[]>` :

```csharp
public class CalculatorTests
{
    [Theory]
    [ClassData(typeof(CalculatorTestData))]
    public void AddReturnsExpectedValueWhenArgsValid(int value1, int value2, int expected)
    {
        var calculator = new Calculator();

        var result = calculator.Add(value1, value2);

        Assert.Equal(expected, result);
    }
}

public class CalculatorTestData : IEnumerable<object[]>
{
    public IEnumerator<object[]> GetEnumerator()
    {
        yield return new object[] { 1, 2, 3 };
        yield return new object[] { -4, -6, -10 };
        yield return new object[] { -2, 2, 0 };
        yield return new object[] { int.MinValue, -1, int.MaxValue };
    }

    IEnumerator IEnumerable.GetEnumerator() => GetEnumerator();
}
```

Source : [https://andrewlock.net/creating-parameterised-tests-in-xunit-with-inlinedata-classdata-and-memberdata/](https://andrewlock.net/creating-parameterised-tests-in-xunit-with-inlinedata-classdata-and-memberdata/).

[^retour en haut](#table-des-matières)

# §5 Dépendances et Mocks

Selon la méthodologie du test unitaire, il faut pouvoir tester un SUT isolément de ses dépendances. Pour ce faire, il faut simuler toutes les dépendances afin de contrôler leur comportement au sein du test, et ainsi ne pas dépendre d'implémentations pouvant interférer avec, voire ralentir, le test du SUT (ceci est réservé aux tests d'intégration).
C'est là que le [framework Moq](https://github.com/Moq/moq4/) entre en jeu. En effet, Moq permet d'instancier des mocks implémentant dynamiquement les méthodes déclarées dans l'interface du SUT à l'aide de quelques lignes de code.

On voit tout de suite qu'il y a un **pré-requis** à l'utilisation de Moq : il est nécessaire que les dépendances à mocker du SUT soient des interfaces injectées par DI, sinon on ne pourra pas utiliser les mocks en lieu et place des implémentations d'origine.
Si ce n'est pas le cas, il faudra créer une interface pour la dépendance posant problème. S'il s'agit d'une dépendance dont on ne peut pas changer le code source (nuget, lib), il faudra l'encapsuler au sein d'un [Adaptateur](<https://fr.wikipedia.org/wiki/Adaptateur_(patron_de_conception)>) qui, lui, implémentera une interface en bonne et due forme.

De même, il est impossible de mocker une classe statique car une classe statique ne peut implémenter une interface. En outre, même s'il est techniquement possible d'utiliser une classe statique sans la mocker (brisant ainsi le principe d'unicité des TU), ceci pourra causer des problèmes d'accès concurrents, les TU s'exécutant en parallèle. Il faut donc à tout prix **proscrire l'utilisation de classes statiques** de type singleton dans les tests.

Les classes de type configuration notamment devront donc être réécrites en tant que services injectables. On pourra garder certaines méthodes et propriétés statiques, si possible privées : elles seront réservées au chargement des données lors de l'initialisation de l'application, mais ne devront pas être testées ni appelées lors des tests. Cf. par exemple le paramètre `UsePortalApi` dans la classe `Settings` de Portal_CPA.

## §5.1 Mocks

Pour mocker une dépendance de type `IDependencyService`, il suffit de faire : `var mock = new Mock<IDependencyService>();`

On peut ensuite mocker la méthode d'`IDependencyService` désirée à l'aide de Setup(), et éventuellement Returns() ou Throws() :

```csharp
var client = _fixture.Create<Client>();
mock.Setup(ds => ds.AddClient(client);   // accepter l'appel à AddClient() avec l'argument client

var clientId = "id1";
mock.Setup(ds => ds.GetClient(clientId)) // à chaque appel à GetClient() avec l'argument cliendId
    .Returns(client);                    // on retourne client

mock.Setup(ds => ds.GetClient(null))                // à chaque appel à GetClient() avec null
    .Throws(new ArgumentNullException("clientId")); // on lève une exception
```

Et en cas de méthode asynchrone :

```csharp
var cardId = "12eaf1-456423-687ea1-fec135";
var cardContent = _fixture.Create<CardContent>();

mock.Setup(ds => ds.GetCardContentAsync(cardId)) // pour une méthode asynchrone renvoyant Task<CardContent>
    .ReturnsAsync(cardContent);                  // on retourne cardContent de manière asynchrone

mock.Setup(ds => ds.UpdateCardContentAsync(cardContent)) // pour une méthode asynchrone renvoyant Task
    .Returns(Task.CompletedTask);                        // on retourne une CompletedTask
```

Pour appeler la méthode mockée, il suffit de faire appel à la dépendance mockée dans la propriété Object :
`var result = await mock.Object.GetCardContentAsync(cardId);`

Pour mocker une propriété :

```csharp
mock.Setup(ds => ds.Name)               // mocke la propriété Name et renvoie systématiquement la même valeur
    .Returns("Olivier");
mock.Setup(ds => ds.ContactCard.Name)   // mocke la propriété Name de la propriété ContactCard (mock récursif)
    .Returns("Aldric");
mock.SetupGet(ds => ds.Name)            // mocke le getter de la propriété Name
    .Returns("Thomas");
mock.SetupSet(ds => ds.Name = "Mehdi"); // mocke le setter de la propriété Name pour n'accepter que la valeur indiquée

mock.SetupProperty(ds => ds.Name);             // bouchonne la propriété Name pour pouvoir l'utiliser ensuite
mock.Object.Name = "Guillaume";
sut.UpdateName(contactCard, mock.Object.Name); // la valeur de la propriété bouchonnée est conservée
contactCard.Name.ShouldBe("Guillaume");
```

Pour stocker (ou logger) l'argument lors d'un appel à une méthode mockée :

```csharp
mock.Setup(ds => ds.UpdateCardContentAsync(cardContent))
    .Callback<CardContent>(cc => receivedArgument = cardContent)
    .Returns(Task.CompletedTask);
```

Pour tester qu'une méthode mockée a bien été appelée (avec l'argument attendu) :

```csharp
mock.Verify(ds => ds.UpdateCardContentAsync(cardContent)); // teste que la méthode UpdateCardContentAsync() a été appelée avec le paramètre cardContent
mock.VerifyAll();                                          // teste que toutes les méthodes spécifiées dans le mock ont été appelées
```

Pour plus d'informations : [Quick start de Moq](https://github.com/Moq/moq4/wiki/Quickstart).

## §5.2 Mock Loose vs Strict

Plusieurs cas de figure se présentent lorsqu'on appelle une méthode du mock :

- Soit la méthode a été mockée et appelée avec les arguments correspondant à ceux définis dans un des Setup() : la valeur de retour correspondante est renvoyée.
- Soit la méthode n'a pas été mockée OU un (ou plusieurs) arguments ne correspondent à aucun de ceux spécifiés dans les Setup() de la méthode mockée : une exception est alors levée (cas du mock Strict) ou une valeur de retour par défaut est renvoyée (cas du mock Loose).

Exemples :

- La méthode est mockée, puis appelée avec un des arguments spécifiés dans le mock (cardId2) :

```csharp
var cardId1 = "12eaf1-456423-687ea1-fec135";
var cardId2 = "eae4f1-879ef3-aef458-00fec5";
var cardContent1 = _fixture.Create<CardContent>();
var cardContent2 = _fixture.Create<CardContent>();

var mock = new Mock<IDependencyService>();

mock.Setup(ds => ds.GetCardContentAsync(cardId1))
    .ReturnsAsync(cardContent1);

mock.Setup(ds => ds.GetCardContentAsync(cardId2))
    .ReturnsAsync(cardContent2);

var result = await mock.Object.GetCardContentAsync(cardId2);
// OK : result == cardContent2 car c'est la valeur de retour qui a été spécifiée dans le mock lors d'un appel avec l'argument cardId2
```

- La méthode n'est pas mockée (mock Loose) :

```csharp
var cardId = "12eaf1-456423-687ea1-fec135";
var cardContent = _fixture.Create<CardContent>();
var updatedCardContent = _fixture.Create<CardContent>();

// Mock Loose
var mock = new Mock<IDependencyService>(MockBehavior.Loose);
// ou simplement : var mock = new Mock<IDependencyService>();

mock.Setup(ds => ds.GetCardContentAsync(cardId))
    .ReturnsAsync(cardContent);

await mock.Object.UpdateCardContentAsync(updatedCardContent);
// OK : le mock Loose autorise les appels aux méthodes de l'interface IDependencyService même si elles ne sont pas mockées spécifiquement,
//      il renvoie simplement la valeur par défaut dans ce cas (ici, Task.CompletedTask qui correspond au void en asynchrone)
```

- La méthode n'est pas mockée (mock Strict) :

```csharp
var cardId = "12eaf1-456423-687ea1-fec135";
var cardContent = _fixture.Create<CardContent>();
var updatedCardContent = _fixture.Create<CardContent>();

// Mock Strict
var mock = new Mock<IDependencyService>(MockBehavior.Strict);

mock.Setup(ds => ds.GetCardContentAsync(cardId))
    .ReturnsAsync(cardContent);

await mock.Object.UpdateCardContentAsync(updatedCardContent);
// KO : une exception est levée car le mock Strict n'autorise pas les appels aux méthodes non mockées
```

- La méthode est mockée, mais appelée avec un argument différent de celui mocké (mock Loose) :

```csharp
var cardId1 = "12eaf1-456423-687ea1-fec135";
var cardId2 = "eae4f1-879ef3-aef458-00fec5";
var cardContent1 = _fixture.Create<CardContent>();
var cardContent2 = _fixture.Create<CardContent>();

// Mock Loose
var mock = new Mock<IDependencyService>(MockBehavior.Loose);
// ou simplement : var mock = new Mock<IDependencyService>();

mock.Setup(ds => ds.GetCardContentAsync(cardId1))
    .ReturnsAsync(cardContent1);
mock.Setup(ds => ds.GetCardContentAsync(cardId2))
    .ReturnsAsync(cardContent2);

var result = await mock.Object.GetCardContentAsync("wrong id");
// OK : le mock Loose accepte les appels avec un argument non spécifié,
//      il renvoie simplement la valeur par défaut dans ce cas (ici, une instance de CarcContent avec tous les champs vides)
```

- La méthode est mockée, mais appelée avec un autre argument que prévu (mock Strict) :

```csharp
var cardId1 = "12eaf1-456423-687ea1-fec135";
var cardId2 = "eae4f1-879ef3-aef458-00fec5";
var cardContent1 = _fixture.Create<CardContent>();
var cardContent2 = _fixture.Create<CardContent>();

// Mock Strict
var mock = new Mock<IDependencyService>(MockBehavior.Strict);

mock.Setup(ds => ds.GetCardContentAsync(cardId1))
    .ReturnsAsync(cardContent1);
mock.Setup(ds => ds.GetCardContentAsync(cardId2))
    .ReturnsAsync(cardContent2);

var result = await mock.Object.GetCardContentAsync("wrong id");
// KO : une exception est levée car le mock Strict n'accepte pas d'appel avec un argument qui n'a pas été spécifié dans le mock
```

On voit donc qu'un mock Strict est utile dans le cas où l'on souhaite verrouiller les méthodes ou propriétés auxquelles le SUT peut accéder. Ceci peut être utile pour vérifier qu'une API ne changeant que très rarement est interrogée correctement et ainsi éviter les régressions.
Cependant, un mock Strict est très contraignant dans le sens où il nous oblige à le mettre à jour à chaque modification importante de la dépendance mockée, ce qui rend les tests fragiles et coûteux à maintenir.

Pour éviter cela, on utilise donc principalement des mocks Loose, qui ne force pas à mocker toutes les méthodes de la dépendance, mais juste celles dont on doit contrôler le comportement pour le TU (valeurs de retour spécifiques, arguments autorisés...).

## §5.3 Filter les arguments (matching)

En général, on n'a pas besoin de spécifier précisément l'argument accepté par une méthode mockée. Pour définir un comportement particulier pour une classe d'arguments plus ou moins précise, on utilise le filtrage ou matching d'arguments.

Par exemple, on accepte tout argument de type CardContent dans la méthode d'update :

```csharp
mock.Setup(ds => ds.UpdateCardContentAsync(It.IsAny<CardContent>()))
    .Returns(Task.CompletedTask);
```

On peut également gérer plusieurs cas, comme ici :

```csharp
// filtrage par prédicat
mock.Setup(ds => ds.UpdateCardContentAsync(Is.Is<CardContent>(cc => cc != null)))
    .Returns(Task.CompletedTask);
mock.Setup(ds => ds.UpdateCardContentAsync(null))
    .Throws<ArgumentNullException>();

// filtrage par intervalle numérique
mock.Setup(ds => ds.UpdateAge(It.IsInRange(0, 130, Range.Inclusive)));

// filtrage par regex.
mock.Setup(ds => ds.UpdateTitle(It.IsRegex("(Mme|Mr|Dr)", RegexOptions.IgnoreCase)));
```

## §5.4 Injection de dépendances et AutoMocking

Pour instancier le service à tester (SUT), le plus simple est de demander à la fixture de la classe de tests mère `SystemUnderTestBase` de le faire :
`var sut = _fixture.Create<IServiceUnderTest>;`

L'implémentation correspondant à l'interface du SUT `IServiceUnderTest` ayant déjà été déclarée dans la fixture (par le constructeur de `SystemUnderTestBase`), elle peut être instanciée.

De même pour les dépendances du SUT, même si elles n'ont pas été spécifiées dans la fixture. En effet, la fonctionnalité d'AutoMocking est également configurée dans la fixture : si un SUT à instancier a des dépendances non spécifiées, elles seront automatiquement mockées avec des mocks par défaut. Il ne reste alors plus qu'à mocker les dépendances dont le comportement par défaut n'est pas suffisant pour tester le SUT dans le contexte désiré.

Pour permettre l'injection d'un mock spécifique, il suffit de le déclarer comme ceci :

```csharp
var customDependencyMock = new Mock<IDependencyService>();
... //

// demande l'injection de l'instance customDependencyMock.Object à chaque instanciation d'un service dépendant de IDependencyService
_fixture.Inject(customDependencyMock.Object);

var sut = _fixture.Create<IServiceUnderTest>();
// => sut._dependencyService == customDependencyMock.Object
```

L'utilisation de l'injection de dépendances et de l'automocking dans les tests contribuent à les rendre moins fragiles à l'ajout ou au retrait de dépendances. Ainsi en cas de refactoring, plus besoin de corriger les instanciations des SUT dans chacun des TU si la méthode testée ne dépend pas directement de la dépendance modifiée.

## §5.5 Classes Mockers

Dans le projet UnitTests, un dossier `CustomMocks` regroupe l'ensemble des classes `Mocker` permettant de générer des mocks pour les différentes dépendances du projet à tester. Ceci permet de factoriser le code des mocks. Ne pas hésiter à enrichir ces classes ou à en créer de nouvelles au besoin.

[^retour en haut](#table-des-matières)

# §6 Exécution des tests et Couverture de code

TODO
[^retour en haut](#table-des-matières)

# §7 TDD et Live testing

TODO
Cf formation qualité développeur ;-).

[^retour en haut](#table-des-matières)

# §8 Liens utiles

- [Xunit](https://xunit.net/#documentation) Le framework d'exécution de tests.
- [Moq](https://github.com/Moq/moq4/wiki) Le framework de mocking (simulation de dépendances).
- [AutoFixture](https://github.com/AutoFixture/AutoFixture/wiki/Cheat-Sheet) Le framework de fixture (génération de données de tests).
- [Shouldly](http://docs.shouldly-lib.net/v2.4.0/docs) Le framework d'assertion.
- [Uno.Equality](https://github.com/nventive/Uno.CodeGen/blob/master/doc/Equality%20Generation.md) Une lib qui génère des méthodes de comparaison d'objets propriété par propriété (et non bêtement par référence). Il suffit juste de rajouter quelques attributs C# à l'entité. Utile pour tester que l'objet renvoyé par la méthode testée est similaire à l'objet attendu.

[^retour en haut](#table-des-matières)
