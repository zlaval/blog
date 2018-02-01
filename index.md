# Tesztelés - kiselőadás 2018.02.01 
* **Unit teszt**
* **Integrációs teszt**
* **Rendszerteszt**

Rendszerteszt (e2e teszt)
-
>A teljes rendszer viselkedését teszteli, az éles rendszerhez hasonló környezetben. Minden komponens és függőség része a tesztnek.
A rendszerteszt elkészítése nem a fejlesztő feladata.

Integrációs teszt
-
>A rendszer komponensei közti együttműködéseket teszteli, vizsgálja a rendszer egészének viselkedését.
Része lehet a teljes rendszer, vagy annak bizonyos részei.

*Például egy REST végpont hívása, mely adatbázisból kiolvasott adatokkal tész vissza.*

JEE környezetben az integrációs tesztek futtatásához szükséges a konténer felépítése. Lehetőség van az alkalmazás szerveren való
futtatásra és tesztelésre. **Arquillian** segítségével akár kiválaszthatjuk, az alkalmazás mely osztályai kerüljenek futtatásra. 
Így alkalmazásszerver segítsége nélkül is futtathatjuk a teszteket, melyeket **JUnit** vagy **TestNG** keretrendszer segítségével készíthetünk el. 


```java
@RunWith(Arquillian.class)
public class DemoEjb {

    @Deployment
    public static JavaArchive createDeployment() {
        return ShrinkWrap.create(JavaArchive.class)
            .addClass(DemoEjb.class)
            .addAsManifestResource(EmptyAsset.INSTANCE, "beans.xml");
    }

    @Test
    public void callSomeFunction() {
        Assert.fail("Not yet implemented");
    }
}
```

REST végpontok tesztelés **REST Assured** segítségével egyszerűen kivitelezhető, köszönhetően a beépített validátoroknak és bejáróknak.

Példa: A ```/lotto/{id}``` végpont a következő _JSON objektumot_ adja válaszként.
```json
{
    "lotto": {
        "lottoId": 5,
        "winning-numbers": [2, 45, 34, 23, 7, 5, 3],
        "winners": [
            {
                "winnerId": 23,
                "numbers": [2, 45, 34, 23, 3, 5]
            }, {
                "winnerId": 54,
                "numbers": [52, 3, 12, 11, 18, 22]
            }
        ]
    }
}
```
Az objektum bejárását és ellenőrzését a keretrendszer elvégzi a megadott útvonal alapján valamint a megadott értékekre.
```java
@Test
public void lottoReturnsWithExpectedIdAndWinners() {
    when()
            .get("/lotto/{id}", 5)
            .then()
            .statusCode(200)
            .body("lotto.lottoId", equalTo(5),
                    "lotto.winners.winnerId", containsOnly(23, 54));
}
```
[Arquillian](http://arquillian.org/)
[Rest-Assured](http://rest-assured.io/)


Unit teszt
-
>A szoftver egységeinek önálló, környezetfüggetlen tesztje. Az üzleti logika egységnyi részeit teszteli.

A unit tesztek három lépésből állnak: 
1. A tesztelendő programrész inicializálása. Ezt a kódrészt hívjuk **SUT**-nak (_system under test_)
2. A programrész végrehajtása
3. Az eredmény kiértékelése

```java
@Test
public void testToLong() {
    //given
    Demo demo=new Demo();
    
    //when
    Long number = demo.toLong("234");
    
    //then
    assertEquals(234L, number);
}
```

Az egységtesztek kiértékelés alapján két fő osztályba sorolhatóak:
1. Állapot alapú. A megfelelő bemenetre az elvárt kimenetet kapjuk eredményül.
2. Viselkedés alapú. A megfelelő metódusok hívódtak a megadott bemenetre.

#### Egységteszt kritériumok:
1. Kicsi, gyors.
2. Környezetfüggetlen. Csak akkor törik, ha a tesztelni kívánt kód hibás.
Gyakori hiba például a tesztek között beragadt állapot.
3. Egy teszt egy assert (csak egy ágat tesztelünk). Könnyebben olvashatóság.
4. Gondosan kiválasztott elnevezés. A teszt nevéből lehessen tudni melyik metódus mely ágát vizsgálja.
5. Külső rendszereket nem tesztel


#### Fontos szempontok

##### NE a lefedettség legyen az első
>A lefedettség növelése egyszerű feladat, de ha a tesztek nem vizsgálják a SUT működésének összes lehetséges ágát,
magas lefedettséggel is sok hiba maradhat a rendszerben.

A következő példa ```100%```-ban lefedett, még sincs megfelelően tesztelve.
```java
public Long toLong(String str) {
    Long number = null;
    if (isNumber(str)) {
        number = Long.valueOf(str);
    }
    return number;
}
```
```java
@Test
public void testToLong() {
    Long number = demo.toLong("234");
    assertEquals(234L, number);
}
```
Hiányzik a negatív ág, amikor számokon kívül más karakterek használatával hajtjuk végre a metódust.

##### Legszigorúbb vizsgálat
>Ha ismerjük a pontos értéket, akkor használjunk **assertEquals**-t **assertNotNull** helyett!

##### Határesetek tesztelése
>Mindíg teszteljük a határeseteket, a legtöbb hiba ott fog előfordulni.
Részletesebben lásd PIT leírásnál
  
##### Negatív ágak tesztelése 
>Teszteljük a lehetséges hiba ágakat, hibás bemenő paramétereket...

##### Használjunk minimális mockolt objektumot
>A túl sok mock jelzi a hibás kódfelépítést. Ilyenkor érdemes újratervezni a kódot.
Ha mindent mockolunk, semmit nem tesztelünk, ami idő és erőforrás pazarló.
Példa: _mapper objektum unit tesztelése rendben van, de webservice hívásé felesleges_.

##### TDD módszertan
>Segít a kód tervezésében, hisz a már kész teszthez alakítjuk az implementációt. Hozzásegít a lazán kapcsolt, 
jól fragmentált rendszer felépítéséhez.

##### SRP
>Egy osztály egy felelősséggel rendelkezzen, így könnyebb tesztelni, valamint függetleníteni más funkcióktól.

##### Dependency Injection
>A függőségek kezelése nem a felhasználó osztály feladata (SRP)

A következő példában egy külső rendszer hívása történik, majd a válasz mappelése megadott formátumra.
A kód nehezen tesztelhető, mivel több felelősséggel rendelkezik és a függőségeket is maga kezeli.
Nehézkes a függőségek cseréje fake objektumra.

```java 
public Response callOuterSystem(RequestData requestData) {
    OuterSystem outerSystem = new OuterSystem();
    OuterSystemResponse outerSystemResponse = outerSystem.call(requestData);
    Response response = new ResponseMapper().conver(outerSystemResponse);
    return response;
}
```
Újratervezve a metódust újrafelhasználható, könnyen tesztelhető és olvashatóbb kódot kapunk.
```java
public OuterSystemResponse callOuterSystem(OuterSystemInterface outerSystem,RequestData requestData){
    //DO SOME OTHER WORK LIKE SET OUTERSYSTEM PROPS
    return outerSystem.call(requestData);
}

public Response mapOuterResponseToResponse(ResponseMapperInterface responseMapper,OuterSystemResponse outerSystemResponse){
    //DO SOME OTHER WORK 
    return  responseMapper.convert(outerSystemResponse)
}
```

##### Kerüljük a külső függőségek mockolását
>Úgy kell kialakítani a kódot, hogy a külső függőségeket lehetőleg ne kelljen mockolni. Mivel bármikor vátozhat egy lib működése,
ha mockoljuk, a függőség verzió váltásánál nem vesszük észre az esetleges hibás működést.

##### Merjünk refaktorálni
>A jól kialakított tesztek mutatják, ha valamit elrontunk.

##### Interfacek használata
>Könnyen cserélhető az implementáció éles és teszt kódban is

##### Állapotmentes kód, minimális függőség

#### Teszt keretrendszerek
A legelterjedtebb Java teszt eszközök a JUnit valamint a Mockito.
Sokak által használt még a PowerMock. Segítégével privát adattagokat tesztelhetünk, statikus metódusokt mockolhatunk.
A használata - nagyon kevés kiváteltől eltekintve - nem ajánlott. Általában hibás tervezést jelent, ha szükség van rá.

##### JUnit
>Biztosítja a tesztek futtatásához, valamint a kimenetek ellenőrzéséhez szükséges funkciókat.


#### Mockito
>Biztosítja a teszt dublőrök létrehozásához szükséges funkcionalitást, valamint az ezeken végzett műveletek stubolását és validálását.
Segítségével megváltoztathatjuk egy függvény visszatérési értékét, megvizsgálhatjuk hányszor hívódott meg, milyen paraméterekkel...

##### Teszt dublőrök:

###### Dummy
>A teszt számára szükséges adatok szimulálása. Pl: Entitások, konfigurációk...
```Person person=new Person("DummyPerson")```

###### Fake
>Egyszerűsített, de működő komponens. Pl: Memórai adatbázis, osztály statikus adatokkal. 
```java
public class FakeAccountRepository implements AccountRepository {

    Map<User, Account> accounts = new HashMap<>();

    public FakeAccountRepository() {
        this.accounts.put(new User("joedoe@gmail.com"), new UserAccount());
        this.accounts.put(new User("gipszjakab@gmail.com"), new AdminAccount());
    }

    String getPasswordHash(User user) {
        return accounts.get(user).getPasswordHash();
    }
}
```
###### Mock/Stub
>Adott osztályból készül objektum váz, mely viselkedését a teszhez igazítottan készíthetjük el. 
```java
//create mock
@Mock private DemoClass demoClass;
//create mock
List mockedList = mock(List.class);

//stubbing
when(mockedList.get(0)).thenReturn("first");
```

###### Spy
>Egy létező példányt wrappel, így megfigyelhetjük az objektum viselkedését. (meghívódott-e a metódus, hányszor..)
```java
List<Integer> spyList = Mockito.spy(new ArrayList());
spyList.add(1);
Mockito.verify(spyList).add(anyInt());
```

#### PIT
Mutációs teszt rendszer, mely segítségével kiszűrhetők a lefedettséget növelő, de hozzáadott értékkel nem rendelkező tesztek.

Az alábbi funkciót teszteljük (az egyszerűség kedvéért a 0-ra true értéket adunk vissza):
```java
public boolean isPositive(int number) {
    if (number >= 0) {
        return true;
    } else {
        return false;
    }
}
```
A teszteset a következő:
```java
@Test
public void testIsPositiveWithPositivNumber() {
    boolean shouldBeTrue = testableClass.isPositive(1);
    assertTrue(shouldBeTrue);
}

@Test
public void testIsPositiveWithNegativeNumber() {
    boolean shouldBeFalse = testableClass.isPositive(-1);
    assertFalse(shouldBeFalse);
}
```
A teszt lefedettség itt is ```100%```, de hiányzik a határ érték (egyenlőség vizsgálata) vizsgálata. A PIT mutáció így elbukik.
A _ConditionalsBoundaryMutator_ mutációval (mely az if kifejezésben a >= jelet > jelre cseréli) 
a teszt továbbra is sikeres.  Négy mutációból 3 **KILLED** (ez azt jelenti, hogy a mutáció után a teszt sikertelen,
tehát jó a teszt, nem verhető át) és 1 **SURVIVED**, tehát sikeres a teszt az isPositive hibás eredménye ellenére.
A tényleges teljes lefedettség eléréséhez vizsgáljuk meg a határ esetet:
```java
@Test
public void testIsPositiveWithZero() {
    boolean shouldBeTrueAgain = testableClass.isPositive(0);
    assertTrue(shouldBeTrueAgain);
}
```
![picture alt](https://github.com/zlaval/Testing/blob/master/pit2.png "Pit result")

[JUnit4](http://junit.org/junit4/)
[JUnit5](http://junit.org/junit5/)
[Mockito](http://site.mockito.org/)
[PIT](http://pitest.org/)

További linkek
---
[TestNG](http://testng.org/doc/)
[EasyMock](http://easymock.org/)
[JMeter](http://jmeter.apache.org/)
[PowerMock](https://github.com/powermock/powermock)

