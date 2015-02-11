## Object.defineProperty() metoda, której mogłeś nie znać z ES5.

Na horyzoncie nowoczesnego programowania pojawiła się nowa wersja języka JavaScript ES6. Dla mnie jednak, jak i wielu innych JavaScript developerów, ES5 ma kilka ciekawych wbudowanych metod, których potencjał nie został jeszcze odkryty. Zdecydowanie warto poświecić więcej czasu na naukę tych mniej popularnych tricków, gdyż są one idealnym rozwiązaniem w momencie, gdy JavaScript nie jest używany tylko do modyfikacji elementów DOM, ale odpowiada też za logikę aplikacji internetowej.

## Problem 
Naszym zadaniem będzie stworzenie konstruktora obiektu __Person__, który otrzyma dwa parametry __imie__, __nazwisko__.
Nasz obiekt będzie miał 4 pola: __firstName__, __lastName__, __fullName__ i __species__. Pierwsze trzy będą reagować na zmiany, a ostatnie - gatunek - będzie niezmienny i otrzyma stałą wartość 'human'.
Zakładamy, że dane są zawsze poprawne i zawsze będą w formacie jednoczłonowego imienia i nazwiska. Przykład niżej:

```JavaScript
var woman = new Person('Jessica', 'Khowalski');

woman.firstName; // 'Jessica'
woman.lastName; // 'Khowalski'
woman.fullName; //'Jessica Khowalski
woman.species; // human

/*
 * Zmnieńmy imię ;-)
 */
 
woman.firstName = 'Fiona';
woman.firstName; // 'Fiona'
woman.lastName; // 'Khowalski'
woman.fullName; // 'Fiona Khowalski' - How to do that !?
woman.species = 'fish';
woman.species; // human - No you can't change this propriety.
 
 /*
  * Zamieńmy pełne imię i nazwisko
  */
  
woman.fullName = 'Joana Stevens';
woman.firstName; // 'Joana'
woman.lastName; // 'Stevens'
```

## Jak to zrobić?

Z pomocą przychodzi nam [Object.defineProperty(obj, prop, descriptor)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty). Jest to metoda ze standardu ECMAScript 5. Metoda służy do definiowania lub nadpisywania atrybutów w obiekcie. 

Jako pierwszy parametr przyjmuję obiekt, w którym będą dodawane nowe atrybuty. Drugi parametr jest nazwą klucza. Trzeci to 'opis' atrybutu. 'Opis' pola jest obiektem, który udostępnia nam następujące pola:

- __configurable__ - opis funkcji (kopiuj wklej z dokumentacji)

- __enumerable__ - opis funkcji (kopiuj wklej z dokumentacji)

- __value__ - opis funkcji (kopiuj wklej z dokumentacji)

- __writable__ - opis funkcji (kopiuj wklej z dokumentacji)

- __get__ - opis funkcji (kopiuj wklej z dokumentacji)

- __set__ - opis funkcji (kopiuj wklej z dokumentacji)

Sprawdźmy zatem, jak rozwiąć zadanie przez zbudowanie konstruktora __Person__, który na razie będzie miał 3 pola __firstName__ , __lastName__ i __species__. Polem __fullName__ zajmiemy się później. 
Jak wynika z dokumentacji _Object.defineProperty_, możemy dodać pole do obiektu i w _descriptor_ ustawić pole __writable__ na _false_, co pozwoli nam uchronić je przed nadpisaniem wartości. Zobaczmy, jak działa to w praktyce:

```JavaScript
var Person = function (first, last) {
  this.firstName = first;
  this.lastName = last;
}

Object.defineProperty(Person, 'species',{
  writable: false,
  value: 'human'
});

var woman = new Person('Jessica', 'Khowalski');
woman; // Person {firstName: 'Jessica', lastName: 'Khowalski', species: 'human'}

/*
 * Try to overwrite species
 */
 
woman.species = 'fish';
woman.species; // 'human' - We can't change this value.
```

Właśnie stworzyliśmy pole, którego nie da się nadpisać. Pierwszy sukces za nami. ;-) 

__*Pro tip nigdy nie używaj metody defineProperty wewnątrz konstruktora*__

Pora na drugą część naszego zadania. Musimy zdefiniować pole __fullName__, które zawsze będzie zwracać nam aktualne imię i nazwisko, a w przypadku nadpisania __fullName__ - pola __firstName__ i __lastName__ zaktualizują się. Używając __get__ i __set__, możemy zdefiniować, co będzie działo się z obiektem i co zostanie zwrócone, gdy odczytamy lub nadpiszemy zdefiniowane pole. Rozwiążmy trudniejszą część naszego zadania, by zobaczyć, jak działa to w praktyce:

```JavaScript
var Person = function (first, last) {
  this.firstName = first;
  this.lastName = last;
};

Object.defineProperty(Person, 'species', {
  writable: false,
    value: 'human'
});

Object.defineProperty(Person, 'fullName', {
  get: function () {
    return this.firstName + ' ' + this.lastName;
  },
  set: function (value) {
    var splitString = value.trim().split(' ');

    if(splitString.length === 2) {
      this.firstName = splitString[0];
      this.lastName = splitString[1];
    }
  }
});

var woman = new Person('Tom', 'Khowalski');

woman.firstName; // 'Jessica'
woman.lastName; // 'Khowalski'
woman.fullName; //'Jessica Khowalski
woman.species; // 'human'

/*
 * Zmnieńmy imię ;-)
 */
 
woman.firstName = 'Fiona';
woman.firstName; // 'Fiona'
woman.lastName; // 'Khowalski'
woman.fullName; // 'Mat Khowalski'
woman.species = 'fish';
woman.species; // 'human' - No, you can't change this properity.
 
 /*
  * Zamieńmy pełne imię i nazwisko
  */
  
woman.fullName = 'Joana Stevens';
woman.firstName; // 'Joana'
woman.lastName; // 'Stevens'
```

## Wsparcie

Object.defineProperty jest wspierany we wszystkich [nowoczesnych przeglądarkach](http://kangax.github.io/compat-table/es5/#Object.defineProperty). Internet Explorer 8 wspiera Object.defineProperty tylko dla obiektów DOM. Warto wspomnieć że Firefox ma [problemy z wydajnością](https://bugzilla.mozilla.org/show_bug.cgi?id=626021) ale odczuwalne jest to dopiero jak mamy więcej niż 100k obiektów.

## Warto wiedzieć

To tylko jedna z wielu metod ES5 zachęcam do zapoznania się z dokumentacją wielu innych ciekawych metod jak:
[Object.defineProperties()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperties) , [Object.freeze()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze),
[Object.keys()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/keys)
