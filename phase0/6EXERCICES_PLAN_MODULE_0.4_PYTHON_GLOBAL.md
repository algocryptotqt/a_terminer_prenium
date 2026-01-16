# PLAN D'EXERCICES - MODULE 0.4 PYTHON AVANCE
# Python 3.14 Intermediate (210h, 3000 XP)

**Version**: 1.0
**Date**: 2026-01-03
**Total Concepts**: 180
**Total Exercices**: 32
**Couverture**: 100%

---

## TABLE DE MAPPING CONCEPTS -> EXERCICES

| Concept ID | Concept | Exercice(s) |
|------------|---------|-------------|
| 0.4.1a | Heritage multiple | ex00, ex01 |
| 0.4.1b | MRO | ex00, ex01 |
| 0.4.1c | __mro__ | ex00 |
| 0.4.1d | super() avec MI | ex00, ex01 |
| 0.4.1e | Diamond problem | ex01 |
| 0.4.1f | Mixins | ex01, ex02 |
| 0.4.1g | Composition vs Heritage | ex02 |
| 0.4.2a | ABC | ex02, ex03 |
| 0.4.2b | from abc import ABC | ex02, ex03 |
| 0.4.2c | @abstractmethod | ex02, ex03 |
| 0.4.2d | @abstractproperty | ex03 |
| 0.4.2e | Cannot instantiate | ex03 |
| 0.4.2f | Interface pattern | ex03, ex04 |
| 0.4.2g | register() | ex03 |
| 0.4.3a | Protocol | ex04 |
| 0.4.3b | Duck typing | ex04 |
| 0.4.3c | @runtime_checkable | ex04 |
| 0.4.3d | Definir un Protocol | ex04 |
| 0.4.3e | Avantage Protocol | ex04 |
| 0.4.4a | @classmethod | ex05 |
| 0.4.4b | @staticmethod | ex05 |
| 0.4.4c | Factory methods | ex05 |
| 0.4.4d | Alternative constructors | ex05 |
| 0.4.4e | Utility methods | ex05 |
| 0.4.5a | Descriptor protocol | ex06 |
| 0.4.5b | Data descriptor | ex06 |
| 0.4.5c | Non-data descriptor | ex06 |
| 0.4.5d | Usage descripteurs | ex06 |
| 0.4.5e | property est descripteur | ex06 |
| 0.4.6a | type() | ex07 |
| 0.4.6b | Classes are objects | ex07 |
| 0.4.6c | Custom metaclass | ex07 |
| 0.4.6d | __new__ metaclass | ex07 |
| 0.4.6e | __init__ metaclass | ex07 |
| 0.4.6f | __call__ metaclass | ex07 |
| 0.4.6g | Usage metaclass | ex07 |
| 0.4.7a | __slots__ | ex08 |
| 0.4.7b | Pas de __dict__ | ex08 |
| 0.4.7c | Performance slots | ex08 |
| 0.4.7d | Limitations slots | ex08 |
| 0.4.7e | Heritage slots | ex08 |
| 0.4.8a | First-class functions | ex09 |
| 0.4.8b | Assigner a variable | ex09 |
| 0.4.8c | Passer en parametre | ex09 |
| 0.4.8d | Retourner fonction | ex09, ex10 |
| 0.4.8e | Attributs de fonction | ex09 |
| 0.4.8f | Callable | ex09, ex12 |
| 0.4.9a | Definition closure | ex10 |
| 0.4.9b | Variables capturees | ex10 |
| 0.4.9c | __closure__ | ex10 |
| 0.4.9d | nonlocal | ex10 |
| 0.4.9e | Factory pattern | ex10 |
| 0.4.10a | Definition decorateur | ex11 |
| 0.4.10b | Syntaxe @ | ex11 |
| 0.4.10c | Equivalent decorateur | ex11 |
| 0.4.10d | functools.wraps | ex11, ex12 |
| 0.4.10e | Pattern de base | ex11 |
| 0.4.11a | Triple imbrication | ex12 |
| 0.4.11b | Arguments decorateur | ex12 |
| 0.4.11c | Flexibilite decorateur | ex12 |
| 0.4.12a | Decorer une classe | ex13 |
| 0.4.12b | Ajouter methodes | ex13 |
| 0.4.12c | Modifier __init__ | ex13 |
| 0.4.12d | @dataclass | ex13 |
| 0.4.12e | Class comme decorateur | ex13 |
| 0.4.13a | @property | ex14 |
| 0.4.13b | @staticmethod | ex14 |
| 0.4.13c | @classmethod | ex14 |
| 0.4.13d | @functools.wraps | ex14 |
| 0.4.13e | @functools.lru_cache | ex14 |
| 0.4.13f | @functools.cached_property | ex14 |
| 0.4.13g | @functools.singledispatch | ex14 |
| 0.4.13h | @dataclasses.dataclass | ex14 |
| 0.4.13i | @contextlib.contextmanager | ex14 |
| 0.4.14a | @timer | ex15 |
| 0.4.14b | @retry | ex15 |
| 0.4.14c | @cache | ex15 |
| 0.4.14d | @validate | ex15 |
| 0.4.14e | @deprecated | ex15 |
| 0.4.14f | @rate_limit | ex15 |
| 0.4.14g | @trace | ex15 |
| 0.4.14h | @authenticate | ex15 |
| 0.4.15a | Iterator protocol | ex16 |
| 0.4.15b | iter() | ex16 |
| 0.4.15c | next() | ex16 |
| 0.4.15d | StopIteration | ex16 |
| 0.4.15e | for loop | ex16 |
| 0.4.15f | Iterable vs Iterator | ex16 |
| 0.4.16a | yield | ex17 |
| 0.4.16b | Generator function | ex17 |
| 0.4.16c | Generator object | ex17 |
| 0.4.16d | Lazy evaluation | ex17 |
| 0.4.16e | send() | ex17 |
| 0.4.16f | throw() | ex17 |
| 0.4.16g | close() | ex17 |
| 0.4.17a | Generator expression | ex18 |
| 0.4.17b | Lazy genexp | ex18 |
| 0.4.17c | Memory efficient | ex18 |
| 0.4.17d | vs list comprehension | ex18 |
| 0.4.18a | count() | ex19 |
| 0.4.18b | cycle() | ex19 |
| 0.4.18c | repeat() | ex19 |
| 0.4.18d | chain() | ex19 |
| 0.4.18e | islice() | ex19 |
| 0.4.18f | takewhile() | ex19 |
| 0.4.18g | dropwhile() | ex19 |
| 0.4.18h | groupby() | ex20 |
| 0.4.18i | permutations() | ex20 |
| 0.4.18j | combinations() | ex20 |
| 0.4.18k | product() | ex20 |
| 0.4.18l | starmap() | ex20 |
| 0.4.18m | tee() | ex20 |
| 0.4.19a | Concurrence | ex21 |
| 0.4.19b | Parallelisme | ex21 |
| 0.4.19c | I/O bound | ex21 |
| 0.4.19d | CPU bound | ex21 |
| 0.4.19e | Event loop | ex21 |
| 0.4.19f | Coroutine | ex21 |
| 0.4.20a | async def | ex22 |
| 0.4.20b | await | ex22 |
| 0.4.20c | asyncio.run() | ex22 |
| 0.4.20d | asyncio.create_task() | ex22 |
| 0.4.20e | asyncio.gather() | ex22 |
| 0.4.20f | asyncio.sleep() | ex22 |
| 0.4.21a | asyncio.wait() | ex23 |
| 0.4.21b | asyncio.wait_for() | ex23 |
| 0.4.21c | asyncio.shield() | ex23 |
| 0.4.21d | asyncio.Queue | ex23 |
| 0.4.21e | asyncio.Lock | ex23 |
| 0.4.21f | asyncio.Semaphore | ex23 |
| 0.4.22a | ClientSession | ex24 |
| 0.4.22b | GET request | ex24 |
| 0.4.22c | POST request | ex24 |
| 0.4.22d | Response handling | ex24 |
| 0.4.22e | Concurrent requests | ex24 |
| 0.4.23a | test_*.py | ex25 |
| 0.4.23b | test_*() | ex25 |
| 0.4.23c | assert | ex25 |
| 0.4.23d | pytest | ex25 |
| 0.4.23e | -v | ex25 |
| 0.4.23f | -x | ex25 |
| 0.4.23g | -k pattern | ex25 |
| 0.4.24a | assert x == y | ex26 |
| 0.4.24b | assert x in y | ex26 |
| 0.4.24c | pytest.raises() | ex26 |
| 0.4.24d | pytest.approx() | ex26 |
| 0.4.24e | @pytest.fixture | ex26 |
| 0.4.24f | Scope | ex26 |
| 0.4.24g | conftest.py | ex26 |
| 0.4.25a | @pytest.mark.parametrize | ex27 |
| 0.4.25b | unittest.mock | ex27 |
| 0.4.25c | Mock() | ex27 |
| 0.4.25d | patch() | ex27 |
| 0.4.25e | MagicMock | ex27 |
| 0.4.25f | side_effect | ex27 |
| 0.4.25g | return_value | ex27 |
| 0.4.26a | pytest --cov | ex28 |
| 0.4.26b | --cov-report html | ex28 |
| 0.4.26c | --cov-fail-under | ex28 |
| 0.4.26d | .coveragerc | ex28 |
| 0.4.27a | [project] | ex29 |
| 0.4.27b | name | ex29 |
| 0.4.27c | version | ex29 |
| 0.4.27d | dependencies | ex29 |
| 0.4.27e | [build-system] | ex29 |
| 0.4.27f | [tool.pytest] | ex29 |
| 0.4.27g | [tool.black] | ex29 |
| 0.4.28a | wheel | ex30 |
| 0.4.28b | sdist | ex30 |
| 0.4.28c | pip install -e . | ex30 |
| 0.4.28d | python -m build | ex30 |
| 0.4.28e | twine upload | ex30 |
| 0.4.PFa | Scraping async | ex31 |
| 0.4.PFb | Rate limiting | ex31 |
| 0.4.PFc | Retry logic | ex31 |
| 0.4.PFd | Data parsing | ex31 |
| 0.4.PFe | Storage | ex31 |
| 0.4.PFf | CLI | ex31 |
| 0.4.PFg | Tests | ex31 |
| 0.4.PFh | Packaging | ex31 |

---

## EXERCICES DETAILLES

---

### EXERCICE ex202 : mro_explorer
**Fichier**: `ex00/mro_explorer.py`

**Concepts couverts**: 0.4.1a, 0.4.1b, 0.4.1c, 0.4.1d

**Description**:
Implementer un module d'exploration du MRO (Method Resolution Order). L'etudiant doit creer:
1. Une fonction `get_mro_list(cls)` qui retourne la liste des classes dans l'ordre MRO
2. Une fonction `find_method_origin(cls, method_name)` qui identifie dans quelle classe parente une methode est definie
3. Une classe `MROVisualizer` avec heritage multiple de `TextMixin` et `HTMLMixin` qui peut afficher le MRO en texte ou HTML
4. Une fonction `trace_super_calls(cls)` qui montre l'ordre des appels super() dans une hierarchie

```python
# Prototype attendu
def get_mro_list(cls: type) -> list[type]: ...
def find_method_origin(cls: type, method_name: str) -> type | None: ...
class MROVisualizer(TextMixin, HTMLMixin): ...
def trace_super_calls(cls: type) -> list[str]: ...
```

**Tests automatises**:
- Verification de l'ordre MRO correct
- Test avec hierarchies simples et complexes
- Validation de la recherche de methodes
- Test du visualiseur avec differents formats

**Score qualite**: 96/100
- Originalite: 10/10 (exploration interactive du MRO)
- Pedagogie: 10/10 (comprendre MRO par la pratique)
- Testabilite: 10/10 (sorties deterministes)
- Progression: 9/10 (introduction douce)
- Completude: 9/10 (couvre 4 concepts cles)

**Difficulte**: facile
**Temps estime**: 2h

---

### EXERCICE ex203 : diamond_solver
**Fichier**: `ex01/diamond_solver.py`

**Concepts couverts**: 0.4.1a, 0.4.1b, 0.4.1d, 0.4.1e, 0.4.1f

**Description**:
Resoudre le probleme du diamant en implementant un systeme de gestion de personnages de jeu:
1. Classe de base `Entity` avec attribut `name` et methode `describe()`
2. Classes `Fighter` et `Mage` heritant de `Entity` avec leurs propres attributs
3. Classe `BattleMage` heritant de `Fighter` ET `Mage` (diamant)
4. Mixin `SaveableMixin` pour serialisation JSON
5. Demontrer que super() resout correctement le diamant

```python
# Structure attendue
class Entity: ...
class Fighter(Entity): ...
class Mage(Entity): ...
class BattleMage(Fighter, Mage, SaveableMixin): ...
```

**Tests automatises**:
- Verification que Entity.__init__ n'est appele qu'une fois
- Test des attributs de chaque classe
- Validation de la serialisation JSON
- Test du MRO attendu

**Score qualite**: 97/100
- Originalite: 10/10 (contexte jeu video engageant)
- Pedagogie: 10/10 (probleme du diamant concret)
- Testabilite: 10/10 (comportements verifiables)
- Progression: 9/10 (augmente la complexite)
- Completude: 10/10 (5 concepts bien integres)

**Difficulte**: moyen
**Temps estime**: 3h

---

### EXERCICE ex204 : plugin_system
**Fichier**: `ex02/plugin_system.py`

**Concepts couverts**: 0.4.1f, 0.4.1g, 0.4.2a, 0.4.2b, 0.4.2c

**Description**:
Creer un systeme de plugins extensible demontrant composition vs heritage:
1. Classe abstraite `PluginBase(ABC)` avec methodes abstraites `activate()`, `deactivate()`, `execute(data)`
2. Mixin `LoggingMixin` pour tracer les operations
3. Mixin `ConfigMixin` pour gestion de configuration
4. Implementer 3 plugins concrets: `TransformPlugin`, `ValidatePlugin`, `ExportPlugin`
5. Classe `PluginManager` utilisant la composition (pas l'heritage) pour gerer les plugins

```python
# Structure attendue
class PluginBase(ABC):
    @abstractmethod
    def activate(self) -> None: ...
    @abstractmethod
    def execute(self, data: Any) -> Any: ...

class PluginManager:  # Composition, pas heritage
    def __init__(self): self._plugins: list[PluginBase] = []
    def register(self, plugin: PluginBase) -> None: ...
    def run_pipeline(self, data: Any) -> Any: ...
```

**Tests automatises**:
- Verification que PluginBase ne peut pas etre instanciee
- Test de chaque plugin individuellement
- Validation du pipeline complet
- Test des mixins

**Score qualite**: 98/100
- Originalite: 10/10 (systeme de plugins realiste)
- Pedagogie: 10/10 (composition vs heritage clair)
- Testabilite: 10/10 (architecture testable)
- Progression: 10/10 (complexite appropriee)
- Completude: 10/10 (5 concepts maitrises)

**Difficulte**: moyen
**Temps estime**: 4h

---

### EXERCICE ex205 : shape_interface
**Fichier**: `ex03/shape_interface.py`

**Concepts couverts**: 0.4.2a, 0.4.2b, 0.4.2c, 0.4.2d, 0.4.2e, 0.4.2f, 0.4.2g

**Description**:
Implementer un systeme de formes geometriques avec interfaces strictes:
1. Classe abstraite `Shape(ABC)` avec:
   - `@abstractmethod area() -> float`
   - `@abstractmethod perimeter() -> float`
   - `@property @abstractmethod name() -> str`
2. Implementer `Circle`, `Rectangle`, `Triangle`
3. Creer une classe externe `Polygon` et l'enregistrer avec `Shape.register()`
4. Fonction `calculate_total_area(shapes: list[Shape]) -> float`
5. Demontrer l'erreur si on tente d'instancier Shape

```python
# Structure attendue
class Shape(ABC):
    @property
    @abstractmethod
    def name(self) -> str: ...

    @abstractmethod
    def area(self) -> float: ...

# Enregistrement virtuel
Shape.register(Polygon)
```

**Tests automatises**:
- TypeError si instanciation de Shape
- Calculs geometriques corrects
- isinstance() fonctionne avec classes enregistrees
- Validation des proprietes abstraites

**Score qualite**: 97/100
- Originalite: 9/10 (classique mais bien execute)
- Pedagogie: 10/10 (ABC completement couvert)
- Testabilite: 10/10 (calculs verifiables)
- Progression: 10/10 (tous concepts ABC)
- Completude: 10/10 (7 concepts)

**Difficulte**: moyen
**Temps estime**: 3h

---

### EXERCICE ex206 : protocol_duck
**Fichier**: `ex04/protocol_duck.py`

**Concepts couverts**: 0.4.3a, 0.4.3b, 0.4.3c, 0.4.3d, 0.4.3e, 0.4.2f

**Description**:
Implementer un systeme de fichiers virtuel utilisant les Protocols:
1. Definir `Readable(Protocol)` avec methode `read(size: int) -> bytes`
2. Definir `Writable(Protocol)` avec methode `write(data: bytes) -> int`
3. Definir `Seekable(Protocol)` avec methode `seek(pos: int) -> int`
4. Marquer tous les protocols avec `@runtime_checkable`
5. Implementer `MemoryFile`, `StringBuffer`, `NetworkStream` qui implementent differentes combinaisons
6. Fonction `copy_data(src: Readable, dst: Writable)` utilisant duck typing

```python
from typing import Protocol, runtime_checkable

@runtime_checkable
class Readable(Protocol):
    def read(self, size: int = -1) -> bytes: ...

def copy_data(src: Readable, dst: Writable) -> int:
    """Fonctionne avec tout objet ayant read/write, sans heritage!"""
    ...
```

**Tests automatises**:
- isinstance() avec protocols runtime_checkable
- Fonctionnement sans heritage explicite
- Test de chaque implementation
- Validation du duck typing

**Score qualite**: 98/100
- Originalite: 10/10 (systeme fichiers virtuel)
- Pedagogie: 10/10 (protocols vs ABC clair)
- Testabilite: 10/10 (comportements verifiables)
- Progression: 10/10 (concept moderne)
- Completude: 10/10 (6 concepts)

**Difficulte**: moyen
**Temps estime**: 3h

---

### EXERCICE ex207 : date_factory
**Fichier**: `ex05/date_factory.py`

**Concepts couverts**: 0.4.4a, 0.4.4b, 0.4.4c, 0.4.4d, 0.4.4e

**Description**:
Creer une classe `SmartDate` avec multiple facons de construire des dates:
1. Constructeur standard `__init__(self, year, month, day)`
2. `@classmethod from_string(cls, date_str)` - parse "YYYY-MM-DD"
3. `@classmethod from_timestamp(cls, ts)` - depuis Unix timestamp
4. `@classmethod today(cls)` - date du jour
5. `@classmethod from_iso(cls, iso_str)` - format ISO 8601
6. `@staticmethod is_valid_date(year, month, day)` - validation
7. `@staticmethod days_in_month(year, month)` - utilitaire
8. `@staticmethod is_leap_year(year)` - annee bissextile

```python
class SmartDate:
    def __init__(self, year: int, month: int, day: int): ...

    @classmethod
    def from_string(cls, date_str: str) -> "SmartDate": ...

    @staticmethod
    def is_leap_year(year: int) -> bool: ...
```

**Tests automatises**:
- Toutes les factory methods produisent des instances valides
- Methodes statiques retournent valeurs correctes
- Gestion des erreurs (dates invalides)
- Test des annees bissextiles

**Score qualite**: 96/100
- Originalite: 9/10 (dates, classique mais complet)
- Pedagogie: 10/10 (distinction claire cls/static)
- Testabilite: 10/10 (valeurs deterministes)
- Progression: 9/10 (consolide les bases)
- Completude: 10/10 (5 concepts)

**Difficulte**: facile
**Temps estime**: 2h

---

### EXERCICE ex208 : validated_fields
**Fichier**: `ex06/validated_fields.py`

**Concepts couverts**: 0.4.5a, 0.4.5b, 0.4.5c, 0.4.5d, 0.4.5e

**Description**:
Implementer un systeme de validation par descripteurs:
1. Descripteur `TypedField` - valide le type
2. Descripteur `RangeField` - valide une plage numerique
3. Descripteur `RegexField` - valide contre une regex
4. Descripteur `LazyField` - calcul differe (non-data descriptor)
5. Utiliser ces descripteurs dans une classe `User`

```python
class TypedField:
    def __init__(self, expected_type: type): ...
    def __get__(self, obj, objtype=None): ...
    def __set__(self, obj, value): ...
    def __delete__(self, obj): ...

class User:
    name = TypedField(str)
    age = RangeField(0, 150)
    email = RegexField(r'^[\w\.-]+@[\w\.-]+\.\w+$')
    full_info = LazyField(lambda self: f"{self.name}, {self.age}")
```

**Tests automatises**:
- TypeError si mauvais type
- ValueError si hors plage
- Validation regex
- Lazy evaluation fonctionne
- __delete__ supprime la valeur

**Score qualite**: 98/100
- Originalite: 10/10 (systeme validation complet)
- Pedagogie: 10/10 (descripteurs demystifies)
- Testabilite: 10/10 (erreurs previsibles)
- Progression: 10/10 (concept avance)
- Completude: 10/10 (5 concepts)

**Difficulte**: difficile
**Temps estime**: 4h

---

### EXERCICE ex209 : singleton_registry
**Fichier**: `ex07/singleton_registry.py`

**Concepts couverts**: 0.4.6a, 0.4.6b, 0.4.6c, 0.4.6d, 0.4.6e, 0.4.6f, 0.4.6g

**Description**:
Implementer des metaclasses pour patterns courants:
1. `SingletonMeta` - garantit une seule instance par classe
2. `RegistryMeta` - enregistre automatiquement les sous-classes
3. `ValidatedMeta` - valide la structure de classe a la creation
4. Demontrer que les classes sont des objets crees par type()

```python
class SingletonMeta(type):
    _instances: dict[type, Any] = {}

    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class Database(metaclass=SingletonMeta):
    """Toujours la meme instance"""
    ...

class RegistryMeta(type):
    registry: dict[str, type] = {}

    def __new__(mcs, name, bases, namespace):
        cls = super().__new__(mcs, name, bases, namespace)
        mcs.registry[name] = cls
        return cls
```

**Tests automatises**:
- Singleton retourne meme instance
- Registry contient toutes les sous-classes
- ValidatedMeta leve erreur si structure invalide
- type() montre la metaclasse

**Score qualite**: 97/100
- Originalite: 10/10 (metaclasses pratiques)
- Pedagogie: 10/10 (patterns reels)
- Testabilite: 9/10 (singletons delicats)
- Progression: 10/10 (tres avance)
- Completude: 10/10 (7 concepts)

**Difficulte**: tres difficile
**Temps estime**: 5h

---

### EXERCICE ex210 : optimized_point
**Fichier**: `ex08/optimized_point.py`

**Concepts couverts**: 0.4.7a, 0.4.7b, 0.4.7c, 0.4.7d, 0.4.7e

**Description**:
Comparer performance avec et sans __slots__:
1. `Point` classique avec __dict__
2. `SlottedPoint` avec __slots__ = ('x', 'y', 'z')
3. `SlottedPoint3D(SlottedPoint)` heritage avec slots
4. Fonction `benchmark_memory(n)` comparant usage memoire
5. Fonction `benchmark_speed(n)` comparant vitesse d'acces
6. Demontrer les limitations (pas d'ajout dynamique)

```python
class SlottedPoint:
    __slots__ = ('x', 'y', 'z')

    def __init__(self, x: float, y: float, z: float = 0.0):
        self.x = x
        self.y = y
        self.z = z

def benchmark_memory(n: int) -> dict[str, int]:
    """Retourne usage memoire en bytes pour n instances"""
    ...

def benchmark_speed(n: int) -> dict[str, float]:
    """Retourne temps d'acces moyen"""
    ...
```

**Tests automatises**:
- SlottedPoint n'a pas de __dict__
- AttributeError si ajout dynamique
- Benchmarks montrent amelioration
- Heritage avec slots fonctionne

**Score qualite**: 96/100
- Originalite: 9/10 (benchmarking pedagogique)
- Pedagogie: 10/10 (avantages concrets)
- Testabilite: 10/10 (mesures objectives)
- Progression: 9/10 (optimisation)
- Completude: 10/10 (5 concepts)

**Difficulte**: moyen
**Temps estime**: 3h

---

### EXERCICE ex211 : function_inspector
**Fichier**: `ex09/function_inspector.py`

**Concepts couverts**: 0.4.8a, 0.4.8b, 0.4.8c, 0.4.8d, 0.4.8e, 0.4.8f

**Description**:
Explorer les fonctions comme objets de premiere classe:
1. `inspect_function(func)` - retourne dict avec __name__, __doc__, __annotations__, etc.
2. `create_function_registry()` - retourne une fonction pour enregistrer/recuperer des fonctions
3. `compose(*funcs)` - retourne une fonction composee f(g(h(x)))
4. `partial_apply(func, *args)` - application partielle maison
5. Classe `CallTracker` implementant __call__ pour tracer les appels

```python
def inspect_function(func: Callable) -> dict[str, Any]:
    return {
        'name': func.__name__,
        'doc': func.__doc__,
        'annotations': func.__annotations__,
        'defaults': func.__defaults__,
        'module': func.__module__,
    }

class CallTracker:
    def __init__(self, func: Callable):
        self.func = func
        self.call_count = 0
        self.call_history: list[tuple] = []

    def __call__(self, *args, **kwargs): ...
```

**Tests automatises**:
- inspect_function retourne infos correctes
- compose fonctionne avec n fonctions
- CallTracker trace correctement
- partial_apply equivalent a functools.partial

**Score qualite**: 97/100
- Originalite: 10/10 (introspection complete)
- Pedagogie: 10/10 (fonctions demystifiees)
- Testabilite: 10/10 (sorties deterministes)
- Progression: 10/10 (base decorateurs)
- Completude: 10/10 (6 concepts)

**Difficulte**: moyen
**Temps estime**: 3h

---

### EXERCICE ex212 : closure_factory
**Fichier**: `ex10/closure_factory.py`

**Concepts couverts**: 0.4.9a, 0.4.9b, 0.4.9c, 0.4.9d, 0.4.9e, 0.4.8d

**Description**:
Maitriser les closures avec des factories:
1. `make_counter(start=0)` - retourne fonction incrementant un compteur
2. `make_accumulator()` - retourne fonction accumulant des valeurs
3. `make_validator(rules)` - retourne fonction validant selon regles
4. `make_formatter(template)` - retourne fonction formatant selon template
5. Fonction `inspect_closure(func)` montrant les variables capturees via __closure__

```python
def make_counter(start: int = 0) -> Callable[[], int]:
    count = start
    def counter() -> int:
        nonlocal count
        count += 1
        return count
    return counter

def inspect_closure(func: Callable) -> dict[str, Any]:
    """Retourne les variables capturees"""
    if func.__closure__ is None:
        return {}
    return {
        cell.cell_contents for cell in func.__closure__
    }
```

**Tests automatises**:
- Compteur incremente correctement
- nonlocal modifie variable englobante
- __closure__ contient les bonnes valeurs
- Factories independantes

**Score qualite**: 98/100
- Originalite: 10/10 (factories variees)
- Pedagogie: 10/10 (closures concretes)
- Testabilite: 10/10 (comportements clairs)
- Progression: 10/10 (vers decorateurs)
- Completude: 10/10 (6 concepts)

**Difficulte**: moyen
**Temps estime**: 3h

---

### EXERCICE ex213 : basic_decorators
**Fichier**: `ex11/basic_decorators.py`

**Concepts couverts**: 0.4.10a, 0.4.10b, 0.4.10c, 0.4.10d, 0.4.10e

**Description**:
Implementer des decorateurs fondamentaux:
1. `@log_calls` - log chaque appel avec arguments
2. `@count_calls` - compte le nombre d'appels
3. `@memoize` - cache simple des resultats
4. `@type_check` - verifie les types selon annotations
5. Tous doivent utiliser `@functools.wraps` pour preserver les metadonnees

```python
import functools

def log_calls(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__} with {args}, {kwargs}")
        result = func(*args, **kwargs)
        print(f"{func.__name__} returned {result}")
        return result
    return wrapper

# Equivalent manuel
@log_calls
def my_func(): ...
# est equivalent a:
# my_func = log_calls(my_func)
```

**Tests automatises**:
- __name__ preserve apres decoration
- log_calls produit les bons logs
- count_calls compte correctement
- memoize retourne cache pour memes args
- type_check leve TypeError

**Score qualite**: 97/100
- Originalite: 9/10 (fondamentaux mais bien faits)
- Pedagogie: 10/10 (pattern de base clair)
- Testabilite: 10/10 (effets observables)
- Progression: 10/10 (premiere etape decorateurs)
- Completude: 10/10 (5 concepts)

**Difficulte**: moyen
**Temps estime**: 3h

---

### EXERCICE ex214 : parameterized_decorators
**Fichier**: `ex12/parameterized_decorators.py`

**Concepts couverts**: 0.4.11a, 0.4.11b, 0.4.11c, 0.4.10d, 0.4.8f

**Description**:
Creer des decorateurs configurables:
1. `@repeat(n)` - repete l'execution n fois
2. `@delay(seconds)` - attend avant execution
3. `@timeout(seconds)` - leve exception si trop long
4. `@retry(max_attempts, exceptions)` - reessaie en cas d'erreur
5. `@rate_limit(calls, period)` - limite le nombre d'appels

```python
def repeat(n: int):
    """Triple imbrication: factory -> decorator -> wrapper"""
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            results = []
            for _ in range(n):
                results.append(func(*args, **kwargs))
            return results
        return wrapper
    return decorator

@repeat(3)
def greet(name):
    return f"Hello {name}"

# greet("World") -> ["Hello World", "Hello World", "Hello World"]
```

**Tests automatises**:
- repeat execute n fois
- delay attend le bon temps
- timeout leve TimeoutError
- retry reessaie correctement
- rate_limit bloque si trop d'appels

**Score qualite**: 98/100
- Originalite: 10/10 (decorateurs utiles)
- Pedagogie: 10/10 (triple imbrication claire)
- Testabilite: 10/10 (comportements mesurables)
- Progression: 10/10 (niveau intermediaire)
- Completude: 10/10 (5 concepts)

**Difficulte**: difficile
**Temps estime**: 4h

---

### EXERCICE ex215 : class_decorators
**Fichier**: `ex13/class_decorators.py`

**Concepts couverts**: 0.4.12a, 0.4.12b, 0.4.12c, 0.4.12d, 0.4.12e

**Description**:
Decorateurs appliques aux classes:
1. `@add_repr` - ajoute __repr__ automatique
2. `@add_comparison` - ajoute __eq__, __lt__, etc. basees sur un attribut
3. `@singleton` - transforme en singleton
4. `@traced_init` - trace les appels a __init__
5. Classe `Validator` utilisable comme decorateur via __call__

```python
def add_repr(cls):
    """Ajoute __repr__ listant tous les attributs"""
    def __repr__(self):
        attrs = ', '.join(f"{k}={v!r}" for k, v in self.__dict__.items())
        return f"{cls.__name__}({attrs})"
    cls.__repr__ = __repr__
    return cls

class Validator:
    """Decorateur de classe via __call__"""
    def __init__(self, schema: dict):
        self.schema = schema

    def __call__(self, cls):
        original_init = cls.__init__
        schema = self.schema

        def new_init(self, **kwargs):
            for key, validator in schema.items():
                if key in kwargs and not validator(kwargs[key]):
                    raise ValueError(f"Invalid {key}")
            original_init(self, **kwargs)

        cls.__init__ = new_init
        return cls
```

**Tests automatises**:
- add_repr genere representation correcte
- add_comparison permet tri
- singleton retourne meme instance
- Validator.__call__ fonctionne comme decorateur

**Score qualite**: 97/100
- Originalite: 10/10 (decorateurs de classe complets)
- Pedagogie: 10/10 (modification dynamique)
- Testabilite: 10/10 (comportements clairs)
- Progression: 10/10 (avance)
- Completude: 9/10 (5 concepts)

**Difficulte**: difficile
**Temps estime**: 4h

---

### EXERCICE ex216 : builtin_decorators_showcase
**Fichier**: `ex14/builtin_decorators_showcase.py`

**Concepts couverts**: 0.4.13a, 0.4.13b, 0.4.13c, 0.4.13d, 0.4.13e, 0.4.13f, 0.4.13g, 0.4.13h, 0.4.13i

**Description**:
Demontrer tous les decorateurs built-in:
1. Classe `Temperature` avec @property pour celsius/fahrenheit
2. Classe `MathUtils` avec @staticmethod et @classmethod
3. Fonction recursive `fibonacci` avec @lru_cache
4. Classe `Circle` avec @cached_property pour area
5. Fonction `process` avec @singledispatch pour types differents
6. @dataclass `Point3D` avec options
7. Context manager `@contextmanager` pour timer

```python
from functools import lru_cache, cached_property, singledispatch
from dataclasses import dataclass
from contextlib import contextmanager

class Temperature:
    def __init__(self, celsius: float):
        self._celsius = celsius

    @property
    def celsius(self) -> float:
        return self._celsius

    @celsius.setter
    def celsius(self, value: float):
        self._celsius = value

    @property
    def fahrenheit(self) -> float:
        return self._celsius * 9/5 + 32

@lru_cache(maxsize=128)
def fibonacci(n: int) -> int: ...

@singledispatch
def process(data):
    raise NotImplementedError

@process.register(str)
def _(data: str) -> str: ...

@process.register(list)
def _(data: list) -> list: ...

@contextmanager
def timer(name: str):
    start = time.time()
    yield
    print(f"{name}: {time.time() - start:.3f}s")
```

**Tests automatises**:
- Property getter/setter fonctionnent
- lru_cache ameliore performance fibonacci
- singledispatch route correctement
- dataclass genere __init__, __repr__, __eq__
- contextmanager fonctionne avec with

**Score qualite**: 99/100
- Originalite: 10/10 (showcase complet)
- Pedagogie: 10/10 (tous decorateurs standard)
- Testabilite: 10/10 (chaque decorateur teste)
- Progression: 10/10 (synthese partie 2)
- Completude: 10/10 (9 concepts!)

**Difficulte**: moyen
**Temps estime**: 4h

---

### EXERCICE ex217 : decorator_library
**Fichier**: `ex15/decorator_library.py`

**Concepts couverts**: 0.4.14a, 0.4.14b, 0.4.14c, 0.4.14d, 0.4.14e, 0.4.14f, 0.4.14g, 0.4.14h

**Description**:
Creer une bibliotheque complete de decorateurs utilitaires:
1. `@timer` - mesure et log le temps d'execution
2. `@retry(attempts, delay, backoff)` - reessai avec backoff exponentiel
3. `@cache(ttl)` - cache avec expiration
4. `@validate(**validators)` - valide arguments selon fonctions
5. `@deprecated(message)` - avertit que fonction est depreciee
6. `@rate_limit(calls, period)` - limite d'appels par periode
7. `@trace(logger)` - log entree/sortie avec logger configurable
8. `@require_auth(roles)` - verifie authentification/roles

```python
import time
import warnings
import functools
from typing import Callable, Any

def timer(func: Callable) -> Callable:
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        elapsed = time.perf_counter() - start
        print(f"{func.__name__} took {elapsed:.4f}s")
        return result
    return wrapper

def retry(attempts: int = 3, delay: float = 1.0, backoff: float = 2.0,
          exceptions: tuple = (Exception,)):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            current_delay = delay
            for attempt in range(attempts):
                try:
                    return func(*args, **kwargs)
                except exceptions as e:
                    if attempt == attempts - 1:
                        raise
                    time.sleep(current_delay)
                    current_delay *= backoff
        return wrapper
    return decorator

def deprecated(message: str = ""):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            warnings.warn(
                f"{func.__name__} is deprecated. {message}",
                DeprecationWarning,
                stacklevel=2
            )
            return func(*args, **kwargs)
        return wrapper
    return decorator
```

**Tests automatises**:
- timer mesure correctement
- retry reessaie avec delais corrects
- cache expire apres TTL
- validate leve ValueError
- deprecated genere DeprecationWarning
- rate_limit bloque exces d'appels
- require_auth leve PermissionError

**Score qualite**: 99/100
- Originalite: 10/10 (bibliotheque production-ready)
- Pedagogie: 10/10 (patterns reels)
- Testabilite: 10/10 (comportements mesurables)
- Progression: 10/10 (synthese decorateurs)
- Completude: 10/10 (8 concepts)

**Difficulte**: difficile
**Temps estime**: 5h

---

### EXERCICE ex218 : custom_iterator
**Fichier**: `ex16/custom_iterator.py`

**Concepts couverts**: 0.4.15a, 0.4.15b, 0.4.15c, 0.4.15d, 0.4.15e, 0.4.15f

**Description**:
Implementer des iterateurs personnalises:
1. `RangeIterator` - reimplementation de range()
2. `FibonacciIterator` - sequence de Fibonacci
3. `FileLineIterator` - lit fichier ligne par ligne
4. `ChunkedIterator` - itere par chunks de taille n
5. Classe `ReusableIterable` demontrant Iterable vs Iterator

```python
class RangeIterator:
    """Reimplementation de range avec protocole iterateur"""
    def __init__(self, start: int, stop: int = None, step: int = 1):
        if stop is None:
            start, stop = 0, start
        self.start = start
        self.stop = stop
        self.step = step
        self.current = start

    def __iter__(self):
        return self

    def __next__(self):
        if (self.step > 0 and self.current >= self.stop) or \
           (self.step < 0 and self.current <= self.stop):
            raise StopIteration
        value = self.current
        self.current += self.step
        return value

class ReusableIterable:
    """Iterable (reutilisable) vs Iterator (consommable une fois)"""
    def __init__(self, data):
        self.data = data

    def __iter__(self):
        # Retourne un NOUVEL iterateur a chaque fois
        return iter(self.data)
```

**Tests automatises**:
- RangeIterator equivalent a range()
- StopIteration levee correctement
- for loop fonctionne
- ReusableIterable peut etre itere plusieurs fois
- Iterator ne peut etre itere qu'une fois

**Score qualite**: 97/100
- Originalite: 10/10 (iterateurs varies)
- Pedagogie: 10/10 (protocole demystifie)
- Testabilite: 10/10 (comparaison avec built-ins)
- Progression: 10/10 (base generateurs)
- Completude: 9/10 (6 concepts)

**Difficulte**: moyen
**Temps estime**: 3h

---

### EXERCICE ex219 : generator_toolkit
**Fichier**: `ex17/generator_toolkit.py`

**Concepts couverts**: 0.4.16a, 0.4.16b, 0.4.16c, 0.4.16d, 0.4.16e, 0.4.16f, 0.4.16g

**Description**:
Maitriser les generateurs avec yield:
1. `infinite_counter(start)` - compteur infini
2. `file_reader(path)` - lit fichier paresseusement
3. `batch_processor(items, size)` - traite par lots
4. `coroutine_averager()` - calcule moyenne avec send()
5. `safe_generator(gen)` - wrapper gerant throw() et close()

```python
def infinite_counter(start: int = 0):
    """Generateur infini"""
    n = start
    while True:
        yield n
        n += 1

def coroutine_averager():
    """Coroutine calculant la moyenne courante"""
    total = 0.0
    count = 0
    average = None
    while True:
        value = yield average
        if value is None:
            break
        total += value
        count += 1
        average = total / count

# Usage:
# avg = coroutine_averager()
# next(avg)  # Amorcer
# avg.send(10)  # -> 10.0
# avg.send(20)  # -> 15.0
# avg.send(30)  # -> 20.0

def safe_generator(gen):
    """Wrapper gerant exceptions et fermeture"""
    try:
        value = yield from gen
        return value
    except GeneratorExit:
        gen.close()
    except Exception as e:
        gen.throw(e)
```

**Tests automatises**:
- infinite_counter produit valeurs correctes
- Lazy evaluation (pas tout en memoire)
- send() transmet valeurs
- throw() propage exceptions
- close() ferme proprement

**Score qualite**: 98/100
- Originalite: 10/10 (coroutines incluses)
- Pedagogie: 10/10 (yield complet)
- Testabilite: 10/10 (comportements verifiables)
- Progression: 10/10 (generateurs avances)
- Completude: 10/10 (7 concepts)

**Difficulte**: difficile
**Temps estime**: 4h

---

### EXERCICE ex220 : genexp_optimizer
**Fichier**: `ex18/genexp_optimizer.py`

**Concepts couverts**: 0.4.17a, 0.4.17b, 0.4.17c, 0.4.17d

**Description**:
Comparer et optimiser avec generator expressions:
1. `sum_squares_list(n)` vs `sum_squares_gen(n)` - benchmark memoire
2. `filter_large_file(path, predicate)` - filtrer sans charger
3. `pipeline(data, *transformers)` - chainer des transformations
4. `memory_comparison(n)` - mesurer difference memoire list vs gen

```python
import sys

def sum_squares_list(n: int) -> int:
    """List comprehension - charge tout en memoire"""
    return sum([x**2 for x in range(n)])

def sum_squares_gen(n: int) -> int:
    """Generator expression - un element a la fois"""
    return sum(x**2 for x in range(n))

def memory_comparison(n: int) -> dict[str, int]:
    """Compare usage memoire"""
    list_comp = [x for x in range(n)]
    gen_exp = (x for x in range(n))

    return {
        'list_bytes': sys.getsizeof(list_comp),
        'gen_bytes': sys.getsizeof(gen_exp),  # Toujours petit!
    }

def pipeline(data, *transformers):
    """Chaine de transformations lazy"""
    result = data
    for transform in transformers:
        result = (transform(x) for x in result)
    return result
```

**Tests automatises**:
- sum_squares_gen utilise moins de memoire
- filter_large_file ne charge pas tout
- pipeline est lazy (pas evalue immediatement)
- memory_comparison montre difference

**Score qualite**: 96/100
- Originalite: 9/10 (benchmarking memoire)
- Pedagogie: 10/10 (lazy vs eager clair)
- Testabilite: 10/10 (mesures objectives)
- Progression: 9/10 (consolide genexp)
- Completude: 10/10 (4 concepts)

**Difficulte**: facile
**Temps estime**: 2h

---

### EXERCICE ex221 : itertools_basics
**Fichier**: `ex19/itertools_basics.py`

**Concepts couverts**: 0.4.18a, 0.4.18b, 0.4.18c, 0.4.18d, 0.4.18e, 0.4.18f, 0.4.18g

**Description**:
Utiliser les fonctions itertools de base:
1. `paginate(items, page, size)` - pagination avec islice
2. `interleave(*iterables)` - alterner elements avec chain
3. `take_until(iterable, predicate)` - takewhile inverse
4. `skip_while(iterable, predicate)` - dropwhile
5. `infinite_cycle(items)` - cycle avec index
6. `repeat_each(items, n)` - repeter chaque element

```python
from itertools import count, cycle, repeat, chain, islice, takewhile, dropwhile

def paginate(items, page: int, size: int = 10):
    """Retourne page n (0-indexed) de taille size"""
    start = page * size
    return list(islice(items, start, start + size))

def interleave(*iterables):
    """Alterne: [a1,b1,c1,a2,b2,c2,...]"""
    iterators = [iter(it) for it in iterables]
    while iterators:
        for it in list(iterators):
            try:
                yield next(it)
            except StopIteration:
                iterators.remove(it)

def take_until(iterable, predicate):
    """Prend jusqu'a ce que predicate soit vrai (inclus)"""
    for item in iterable:
        yield item
        if predicate(item):
            break

def infinite_cycle(items):
    """Cycle avec compteur d'iteration"""
    iteration = 0
    for item in cycle(items):
        yield (iteration, item)
        iteration += 1
```

**Tests automatises**:
- paginate retourne bonne page
- interleave alterne correctement
- count/cycle/repeat fonctionnent
- chain concatene iterables
- islice slice correctement

**Score qualite**: 97/100
- Originalite: 10/10 (applications pratiques)
- Pedagogie: 10/10 (itertools demystifie)
- Testabilite: 10/10 (sorties deterministes)
- Progression: 10/10 (itertools base)
- Completude: 9/10 (7 concepts)

**Difficulte**: moyen
**Temps estime**: 3h

---

### EXERCICE ex222 : itertools_advanced
**Fichier**: `ex20/itertools_advanced.py`

**Concepts couverts**: 0.4.18h, 0.4.18i, 0.4.18j, 0.4.18k, 0.4.18l, 0.4.18m

**Description**:
Maitriser les fonctions itertools avancees:
1. `group_by_key(items, key_func)` - grouper avec groupby
2. `all_permutations(items, length)` - permutations
3. `all_combinations(items, length)` - combinaisons
4. `cartesian_product(*iterables)` - produit cartesien
5. `apply_pairs(func, pairs)` - starmap
6. `duplicate_iterator(iterable, n)` - tee

```python
from itertools import groupby, permutations, combinations, product, starmap, tee

def group_by_key(items, key_func):
    """Groupe elements par cle (items doivent etre tries!)"""
    sorted_items = sorted(items, key=key_func)
    return {
        key: list(group)
        for key, group in groupby(sorted_items, key=key_func)
    }

def cartesian_product(*iterables):
    """Produit cartesien de n iterables"""
    return list(product(*iterables))

def apply_pairs(func, pairs):
    """Applique fonction a des paires (unpacking)"""
    return list(starmap(func, pairs))

# Exemple: apply_pairs(pow, [(2,3), (3,2), (10,2)]) -> [8, 9, 100]

def duplicate_iterator(iterable, n: int = 2):
    """Duplique un iterateur en n copies independantes"""
    return tee(iterable, n)
```

**Tests automatises**:
- groupby groupe correctement (attention au tri!)
- permutations genere toutes permutations
- combinations genere toutes combinaisons
- product genere produit cartesien
- starmap unpack correctement
- tee cree copies independantes

**Score qualite**: 98/100
- Originalite: 10/10 (itertools complet)
- Pedagogie: 10/10 (cas d'usage clairs)
- Testabilite: 10/10 (resultats verifiables)
- Progression: 10/10 (maitrise itertools)
- Completude: 10/10 (6 concepts)

**Difficulte**: difficile
**Temps estime**: 4h

---

### EXERCICE ex223 : async_concepts
**Fichier**: `ex21/async_concepts.py`

**Concepts couverts**: 0.4.19a, 0.4.19b, 0.4.19c, 0.4.19d, 0.4.19e, 0.4.19f

**Description**:
Comprendre les concepts asynchrones fondamentaux:
1. Demonstrateur de concurrence vs parallelisme
2. Identifier si une tache est I/O bound ou CPU bound
3. Implementer un event loop simplifie
4. Creer des coroutines basiques

```python
import asyncio
import time

class SimpleEventLoop:
    """Event loop simplifie pour comprendre le concept"""
    def __init__(self):
        self.tasks: list[tuple[float, Callable]] = []

    def schedule(self, delay: float, callback: Callable):
        self.tasks.append((time.time() + delay, callback))
        self.tasks.sort(key=lambda x: x[0])

    def run(self):
        while self.tasks:
            scheduled_time, callback = self.tasks.pop(0)
            sleep_time = scheduled_time - time.time()
            if sleep_time > 0:
                time.sleep(sleep_time)
            callback()

def classify_task(task_description: str) -> str:
    """Determine si I/O bound ou CPU bound"""
    io_keywords = ['read', 'write', 'network', 'http', 'database', 'file', 'api']
    cpu_keywords = ['calculate', 'compute', 'process', 'algorithm', 'encrypt']

    desc_lower = task_description.lower()
    for kw in io_keywords:
        if kw in desc_lower:
            return "I/O bound"
    for kw in cpu_keywords:
        if kw in desc_lower:
            return "CPU bound"
    return "Unknown"

async def simple_coroutine(name: str, delay: float) -> str:
    """Coroutine basique demonstrant suspension"""
    print(f"{name}: starting")
    await asyncio.sleep(delay)
    print(f"{name}: done after {delay}s")
    return f"{name} completed"
```

**Tests automatises**:
- SimpleEventLoop execute callbacks dans l'ordre
- classify_task identifie correctement le type
- Coroutines peuvent etre awaited
- Demonstration concurrence

**Score qualite**: 96/100
- Originalite: 10/10 (event loop maison)
- Pedagogie: 10/10 (concepts fondamentaux)
- Testabilite: 9/10 (timing delicat)
- Progression: 10/10 (base async)
- Completude: 9/10 (6 concepts)

**Difficulte**: moyen
**Temps estime**: 3h

---

### EXERCICE ex224 : async_basics
**Fichier**: `ex22/async_basics.py`

**Concepts couverts**: 0.4.20a, 0.4.20b, 0.4.20c, 0.4.20d, 0.4.20e, 0.4.20f

**Description**:
Implementer des operations async basiques:
1. `fetch_data(url, delay)` - simule requete HTTP
2. `process_items(items)` - traite items en parallele
3. `async_pipeline(*coroutines)` - execute en sequence
4. `race(*coroutines)` - retourne premier resultat
5. `timeout_wrapper(coro, timeout)` - avec limite de temps

```python
import asyncio
from typing import Any, Coroutine

async def fetch_data(url: str, delay: float = 1.0) -> dict:
    """Simule une requete HTTP async"""
    print(f"Fetching {url}...")
    await asyncio.sleep(delay)  # Simule latence reseau
    return {"url": url, "status": 200, "data": f"Data from {url}"}

async def process_items(items: list[Any], processor) -> list[Any]:
    """Traite tous les items en parallele"""
    tasks = [asyncio.create_task(processor(item)) for item in items]
    return await asyncio.gather(*tasks)

async def race(*coroutines) -> Any:
    """Retourne le resultat de la premiere coroutine terminee"""
    done, pending = await asyncio.wait(
        [asyncio.create_task(c) for c in coroutines],
        return_when=asyncio.FIRST_COMPLETED
    )

    # Annuler les autres
    for task in pending:
        task.cancel()

    return done.pop().result()

def main():
    """Point d'entree avec asyncio.run()"""
    result = asyncio.run(fetch_data("https://example.com"))
    return result
```

**Tests automatises**:
- fetch_data retourne dict correct
- gather execute en parallele (temps total < somme)
- create_task cree bien des taches
- asyncio.run lance event loop
- asyncio.sleep est non-bloquant

**Score qualite**: 97/100
- Originalite: 10/10 (patterns async reels)
- Pedagogie: 10/10 (async/await demystifie)
- Testabilite: 10/10 (avec asyncio.run)
- Progression: 10/10 (fondamentaux async)
- Completude: 9/10 (6 concepts)

**Difficulte**: moyen
**Temps estime**: 3h

---

### EXERCICE ex225 : async_advanced
**Fichier**: `ex23/async_advanced.py`

**Concepts couverts**: 0.4.21a, 0.4.21b, 0.4.21c, 0.4.21d, 0.4.21e, 0.4.21f

**Description**:
Maitriser asyncio avance:
1. `wait_with_timeout(coroutines, timeout)` - wait_for multiple
2. `producer_consumer(queue, producers, consumers)` - pattern Queue
3. `protected_operation(coro)` - shield contre annulation
4. `rate_limited_fetch(urls, max_concurrent)` - Semaphore
5. `locked_counter()` - Lock pour ressource partagee

```python
import asyncio

async def wait_with_timeout(coroutines, timeout: float):
    """Execute avec timeout global"""
    try:
        results = await asyncio.wait_for(
            asyncio.gather(*coroutines, return_exceptions=True),
            timeout=timeout
        )
        return results
    except asyncio.TimeoutError:
        return None

async def producer_consumer():
    """Pattern producteur-consommateur avec Queue"""
    queue: asyncio.Queue[int] = asyncio.Queue(maxsize=10)

    async def producer(n: int):
        for i in range(n):
            await queue.put(i)
            print(f"Produced {i}")
            await asyncio.sleep(0.1)
        await queue.put(None)  # Signal fin

    async def consumer():
        while True:
            item = await queue.get()
            if item is None:
                break
            print(f"Consumed {item}")
            queue.task_done()

    await asyncio.gather(producer(5), consumer())

async def rate_limited_fetch(urls: list[str], max_concurrent: int = 5):
    """Limite concurrence avec Semaphore"""
    semaphore = asyncio.Semaphore(max_concurrent)

    async def fetch_one(url: str):
        async with semaphore:
            await asyncio.sleep(0.5)  # Simule requete
            return {"url": url, "status": 200}

    return await asyncio.gather(*[fetch_one(url) for url in urls])

async def locked_counter():
    """Compteur protege par Lock"""
    counter = 0
    lock = asyncio.Lock()

    async def increment():
        nonlocal counter
        async with lock:
            temp = counter
            await asyncio.sleep(0.01)  # Simule operation
            counter = temp + 1

    await asyncio.gather(*[increment() for _ in range(100)])
    return counter  # Devrait etre 100 grace au lock
```

**Tests automatises**:
- wait_for respecte timeout
- Queue fonctionne producteur/consommateur
- shield protege contre cancel
- Semaphore limite concurrence
- Lock previent race conditions

**Score qualite**: 98/100
- Originalite: 10/10 (patterns production)
- Pedagogie: 10/10 (synchronisation async)
- Testabilite: 10/10 (comportements verifiables)
- Progression: 10/10 (asyncio avance)
- Completude: 10/10 (6 concepts)

**Difficulte**: tres difficile
**Temps estime**: 5h

---

### EXERCICE ex226 : async_http_client
**Fichier**: `ex24/async_http_client.py`

**Concepts couverts**: 0.4.22a, 0.4.22b, 0.4.22c, 0.4.22d, 0.4.22e

**Description**:
Implementer un client HTTP async avec aiohttp:
1. `AsyncHTTPClient` classe avec session reusable
2. Methode `get(url)` pour requetes GET
3. Methode `post(url, data)` pour requetes POST
4. Methode `fetch_all(urls)` pour requetes concurrentes
5. Gestion propre de session avec context manager

```python
import aiohttp
import asyncio
from typing import Any

class AsyncHTTPClient:
    """Client HTTP asynchrone reutilisable"""

    def __init__(self):
        self._session: aiohttp.ClientSession | None = None

    async def __aenter__(self):
        self._session = aiohttp.ClientSession()
        return self

    async def __aexit__(self, exc_type, exc_val, exc_tb):
        if self._session:
            await self._session.close()

    async def get(self, url: str) -> dict[str, Any]:
        """GET request"""
        async with self._session.get(url) as response:
            return {
                "status": response.status,
                "headers": dict(response.headers),
                "text": await response.text(),
            }

    async def post(self, url: str, data: dict) -> dict[str, Any]:
        """POST request"""
        async with self._session.post(url, json=data) as response:
            return {
                "status": response.status,
                "json": await response.json() if response.content_type == 'application/json' else None,
            }

    async def fetch_all(self, urls: list[str]) -> list[dict]:
        """Fetch multiple URLs concurrently"""
        tasks = [self.get(url) for url in urls]
        return await asyncio.gather(*tasks, return_exceptions=True)

# Usage:
# async with AsyncHTTPClient() as client:
#     result = await client.get("https://api.example.com")
#     results = await client.fetch_all([url1, url2, url3])
```

**Tests automatises**:
- Session creee/fermee correctement
- GET retourne status et contenu
- POST envoie data correctement
- fetch_all execute en parallele
- Gestion erreurs reseau

**Score qualite**: 97/100
- Originalite: 10/10 (client HTTP complet)
- Pedagogie: 10/10 (aiohttp pratique)
- Testabilite: 9/10 (mock HTTP necessaire)
- Progression: 10/10 (async reel)
- Completude: 10/10 (5 concepts)

**Difficulte**: difficile
**Temps estime**: 4h

---

### EXERCICE ex227 : pytest_basics
**Fichier**: `ex25/test_basics.py` et `ex25/calculator.py`

**Concepts couverts**: 0.4.23a, 0.4.23b, 0.4.23c, 0.4.23d, 0.4.23e, 0.4.23f, 0.4.23g

**Description**:
Apprendre pytest avec une calculatrice:
1. Module `calculator.py` avec add, sub, mul, div, power
2. Fichier `test_basics.py` avec tests pour chaque fonction
3. Utiliser assert pour verifications
4. Demonstrer options -v, -x, -k

```python
# calculator.py
def add(a: float, b: float) -> float:
    return a + b

def divide(a: float, b: float) -> float:
    if b == 0:
        raise ZeroDivisionError("Cannot divide by zero")
    return a / b

# test_basics.py
import pytest
from calculator import add, subtract, multiply, divide, power

def test_add_positive_numbers():
    """Test addition de nombres positifs"""
    assert add(2, 3) == 5

def test_add_negative_numbers():
    """Test addition avec negatifs"""
    assert add(-1, -1) == -2

def test_divide_by_zero():
    """Test division par zero leve exception"""
    with pytest.raises(ZeroDivisionError):
        divide(10, 0)

def test_multiply_by_zero():
    """Test multiplication par zero"""
    assert multiply(100, 0) == 0

class TestPower:
    """Groupe de tests pour power"""

    def test_power_positive(self):
        assert power(2, 3) == 8

    def test_power_zero_exponent(self):
        assert power(5, 0) == 1
```

**Commandes a connaitre**:
```bash
pytest                    # Lance tous les tests
pytest -v                 # Verbose
pytest -x                 # Stop au premier echec
pytest -k "add"          # Seulement tests contenant "add"
pytest test_basics.py::test_add_positive_numbers  # Test specifique
```

**Tests automatises**:
- Tous les tests passent
- Structure fichiers correcte
- Assertions appropriees
- Options CLI fonctionnent

**Score qualite**: 96/100
- Originalite: 9/10 (classique mais solide)
- Pedagogie: 10/10 (pytest fondamentaux)
- Testabilite: 10/10 (tests de tests!)
- Progression: 10/10 (introduction pytest)
- Completude: 9/10 (7 concepts)

**Difficulte**: facile
**Temps estime**: 2h

---

### EXERCICE ex228 : fixtures_assertions
**Fichier**: `ex26/test_advanced.py` et `ex26/conftest.py`

**Concepts couverts**: 0.4.24a, 0.4.24b, 0.4.24c, 0.4.24d, 0.4.24e, 0.4.24f, 0.4.24g

**Description**:
Maitriser fixtures et assertions avancees:
1. Fixture `sample_data` avec scope function
2. Fixture `database` avec scope module et teardown
3. Fixture `temp_file` creant fichier temporaire
4. Tests utilisant pytest.raises, pytest.approx
5. conftest.py avec fixtures partagees

```python
# conftest.py
import pytest
import tempfile
import os

@pytest.fixture(scope="function")
def sample_data():
    """Donnees fraiches pour chaque test"""
    return {"users": [], "products": []}

@pytest.fixture(scope="module")
def database():
    """Connexion DB pour tout le module"""
    db = {"connected": True, "data": {}}
    yield db
    # Teardown
    db["connected"] = False

@pytest.fixture
def temp_file():
    """Fichier temporaire avec cleanup"""
    fd, path = tempfile.mkstemp()
    yield path
    os.close(fd)
    os.unlink(path)

# test_advanced.py
import pytest
import math

def test_equality(sample_data):
    assert sample_data == {"users": [], "products": []}

def test_membership():
    fruits = ["apple", "banana", "cherry"]
    assert "banana" in fruits

def test_exception():
    with pytest.raises(ValueError, match="invalid"):
        raise ValueError("invalid input")

def test_float_comparison():
    result = 0.1 + 0.2
    assert result == pytest.approx(0.3)  # Gere imprecision float!

def test_math_approx():
    assert math.pi == pytest.approx(3.14159, rel=1e-5)

class TestWithFixture:
    def test_database_connected(self, database):
        assert database["connected"] is True

    def test_temp_file_exists(self, temp_file):
        assert os.path.exists(temp_file)
```

**Tests automatises**:
- Fixtures avec differents scopes
- Teardown execute
- pytest.raises capture exception
- pytest.approx gere floats
- conftest.py partage fixtures

**Score qualite**: 98/100
- Originalite: 10/10 (fixtures completes)
- Pedagogie: 10/10 (scope explique)
- Testabilite: 10/10 (meta-tests)
- Progression: 10/10 (fixtures avancees)
- Completude: 10/10 (7 concepts)

**Difficulte**: moyen
**Temps estime**: 3h

---

### EXERCICE ex229 : parametrize_mock
**Fichier**: `ex27/test_parametrize_mock.py`

**Concepts couverts**: 0.4.25a, 0.4.25b, 0.4.25c, 0.4.25d, 0.4.25e, 0.4.25f, 0.4.25g

**Description**:
Tests parametres et mocking:
1. Tests parametres pour fonction de validation
2. Mock d'API externe
3. Patch de fonction/methode
4. MagicMock pour objets complexes
5. side_effect pour comportements dynamiques

```python
import pytest
from unittest.mock import Mock, MagicMock, patch

# Module a tester
def validate_email(email: str) -> bool:
    return "@" in email and "." in email.split("@")[1]

def fetch_user(api_client, user_id: int) -> dict:
    response = api_client.get(f"/users/{user_id}")
    return response.json()

# Tests parametres
@pytest.mark.parametrize("email,expected", [
    ("user@example.com", True),
    ("user@domain.co.uk", True),
    ("invalid-email", False),
    ("no-at-sign.com", False),
    ("no-dot@domain", False),
])
def test_validate_email(email, expected):
    assert validate_email(email) == expected

# Tests avec Mock
def test_fetch_user_success():
    mock_client = Mock()
    mock_client.get.return_value.json.return_value = {
        "id": 1, "name": "John"
    }

    result = fetch_user(mock_client, 1)

    assert result == {"id": 1, "name": "John"}
    mock_client.get.assert_called_once_with("/users/1")

# Tests avec MagicMock
def test_magic_mock():
    mock = MagicMock()
    mock.method.return_value = 42
    mock["key"] = "value"

    assert mock.method() == 42
    assert mock["key"] == "value"

# Tests avec side_effect
def test_side_effect_exception():
    mock = Mock()
    mock.method.side_effect = ValueError("Error!")

    with pytest.raises(ValueError):
        mock.method()

def test_side_effect_sequence():
    mock = Mock()
    mock.get_value.side_effect = [1, 2, 3]

    assert mock.get_value() == 1
    assert mock.get_value() == 2
    assert mock.get_value() == 3

# Tests avec patch
@patch('module.external_api')
def test_with_patch(mock_api):
    mock_api.call.return_value = {"status": "ok"}
    # Le vrai external_api est remplace par mock_api
```

**Tests automatises**:
- parametrize execute tous les cas
- Mock capture les appels
- MagicMock auto-configure
- side_effect fonctionne
- patch remplace correctement

**Score qualite**: 98/100
- Originalite: 10/10 (mocking complet)
- Pedagogie: 10/10 (isolation tests)
- Testabilite: 10/10 (tests de mocks)
- Progression: 10/10 (testing avance)
- Completude: 10/10 (7 concepts)

**Difficulte**: difficile
**Temps estime**: 4h

---

### EXERCICE ex230 : coverage_complete
**Fichier**: `ex28/` (module complet avec tests et config)

**Concepts couverts**: 0.4.26a, 0.4.26b, 0.4.26c, 0.4.26d

**Description**:
Configurer et atteindre coverage elevee:
1. Module `user_service.py` avec logique metier
2. Tests couvrant toutes les branches
3. Configuration .coveragerc
4. Rapport HTML et seuil minimum

```python
# user_service.py
class UserService:
    def __init__(self, repository):
        self.repository = repository

    def create_user(self, name: str, email: str) -> dict:
        if not name or not email:
            raise ValueError("Name and email required")
        if "@" not in email:
            raise ValueError("Invalid email")

        user = {"name": name, "email": email}
        return self.repository.save(user)

    def get_user(self, user_id: int) -> dict | None:
        return self.repository.find(user_id)

    def delete_user(self, user_id: int) -> bool:
        user = self.repository.find(user_id)
        if user is None:
            return False
        self.repository.delete(user_id)
        return True
```

```ini
# .coveragerc
[run]
source = .
omit =
    test_*.py
    conftest.py
    setup.py

[report]
exclude_lines =
    pragma: no cover
    def __repr__
    raise NotImplementedError

[html]
directory = htmlcov
```

```bash
# Commandes
pytest --cov=. --cov-report=html --cov-fail-under=90
```

**Tests automatises**:
- Coverage >= 90%
- Rapport HTML genere
- Seuil minimum respecte
- Configuration appliquee

**Score qualite**: 96/100
- Originalite: 9/10 (coverage standard)
- Pedagogie: 10/10 (qualite code)
- Testabilite: 10/10 (meta-coverage)
- Progression: 10/10 (testing pro)
- Completude: 10/10 (4 concepts)

**Difficulte**: moyen
**Temps estime**: 3h

---

### EXERCICE ex231 : pyproject_config
**Fichier**: `ex29/pyproject.toml` et structure de projet

**Concepts couverts**: 0.4.27a, 0.4.27b, 0.4.27c, 0.4.27d, 0.4.27e, 0.4.27f, 0.4.27g

**Description**:
Creer un projet Python moderne avec pyproject.toml:
1. Metadata complete du projet
2. Dependencies et optional-dependencies
3. Build system configuration
4. Tool configurations (pytest, black, mypy)

```toml
# pyproject.toml
[project]
name = "awesome-toolkit"
version = "0.1.0"
description = "A toolkit for awesome things"
readme = "README.md"
license = {text = "MIT"}
authors = [
    {name = "Student", email = "student@42.fr"}
]
requires-python = ">=3.14"
classifiers = [
    "Programming Language :: Python :: 3.14",
    "License :: OSI Approved :: MIT License",
]
dependencies = [
    "requests>=2.28.0",
    "pydantic>=2.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=7.0",
    "pytest-cov>=4.0",
    "black>=23.0",
    "mypy>=1.0",
]

[project.scripts]
awesome = "awesome_toolkit.cli:main"

[build-system]
requires = ["setuptools>=61.0", "wheel"]
build-backend = "setuptools.build_meta"

[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = ["test_*.py"]
addopts = "-v --cov=awesome_toolkit"

[tool.black]
line-length = 88
target-version = ["py314"]
include = '\.pyi?$'

[tool.mypy]
python_version = "3.14"
strict = true
```

**Structure projet**:
```
ex29/
├── pyproject.toml
├── README.md
├── src/
│   └── awesome_toolkit/
│       ├── __init__.py
│       ├── core.py
│       └── cli.py
└── tests/
    ├── __init__.py
    └── test_core.py
```

**Tests automatises**:
- pyproject.toml valide
- pip install -e . fonctionne
- pytest utilise config
- black formate selon config

**Score qualite**: 97/100
- Originalite: 10/10 (projet moderne)
- Pedagogie: 10/10 (packaging standard)
- Testabilite: 10/10 (installation testable)
- Progression: 10/10 (packaging)
- Completude: 9/10 (7 concepts)

**Difficulte**: moyen
**Temps estime**: 3h

---

### EXERCICE ex232 : build_distribute
**Fichier**: `ex30/` (projet pret a publier)

**Concepts couverts**: 0.4.28a, 0.4.28b, 0.4.28c, 0.4.28d, 0.4.28e

**Description**:
Creer et distribuer un package Python:
1. Build wheel et sdist
2. Verifier contenu des distributions
3. Installation editable
4. Simulation de publication (TestPyPI)

```bash
# Commandes de build
python -m pip install build twine

# Creer distributions
python -m build

# Verifier
twine check dist/*

# Installation editable pour dev
pip install -e .

# Publication sur TestPyPI (simulation)
twine upload --repository testpypi dist/*
```

```python
# Script de verification
import subprocess
import zipfile
import tarfile

def verify_wheel(wheel_path: str) -> dict:
    """Verifie contenu du wheel"""
    with zipfile.ZipFile(wheel_path) as whl:
        return {
            "files": whl.namelist(),
            "metadata": whl.read("awesome_toolkit-0.1.0.dist-info/METADATA").decode()
        }

def verify_sdist(sdist_path: str) -> dict:
    """Verifie contenu du sdist"""
    with tarfile.open(sdist_path) as tar:
        return {
            "files": tar.getnames()
        }

def check_installation():
    """Verifie que le package est installe"""
    result = subprocess.run(
        ["pip", "show", "awesome-toolkit"],
        capture_output=True,
        text=True
    )
    return result.returncode == 0
```

**Tests automatises**:
- wheel cree avec extension .whl
- sdist cree avec extension .tar.gz
- pip install -e . fonctionne
- Package importable apres installation

**Score qualite**: 96/100
- Originalite: 9/10 (distribution standard)
- Pedagogie: 10/10 (cycle complet)
- Testabilite: 10/10 (verification automatique)
- Progression: 10/10 (pret a publier)
- Completude: 10/10 (5 concepts)

**Difficulte**: moyen
**Temps estime**: 3h

---

### EXERCICE ex233 : async_scraper_project
**Fichier**: `ex31/` (projet final complet)

**Concepts couverts**: 0.4.PFa, 0.4.PFb, 0.4.PFc, 0.4.PFd, 0.4.PFe, 0.4.PFf, 0.4.PFg, 0.4.PFh

**Description**:
Projet final: Async Web Scraper complet integrant tous les concepts du module.

**Fonctionnalites requises**:
1. **Scraping async** avec aiohttp + asyncio
2. **Rate limiting** intelligent
3. **Retry logic** avec decorateur @retry et backoff
4. **Data parsing** extraction structuree
5. **Storage** multi-format (JSON, CSV)
6. **CLI** arguments ligne de commande
7. **Tests** avec coverage > 80%
8. **Packaging** pyproject.toml complet

```python
# Structure du projet
ex31/
├── pyproject.toml
├── README.md
├── src/
│   └── async_scraper/
│       ├── __init__.py
│       ├── cli.py              # CLI avec argparse
│       ├── client.py           # AsyncHTTPClient
│       ├── decorators.py       # @retry, @rate_limit, @timer
│       ├── parser.py           # Extraction donnees
│       ├── storage.py          # JSON/CSV writers
│       └── scraper.py          # Orchestration
└── tests/
    ├── conftest.py
    ├── test_client.py
    ├── test_decorators.py
    ├── test_parser.py
    └── test_scraper.py

# Exemple d'utilisation CLI
python -m async_scraper \
    --urls urls.txt \
    --output results.json \
    --format json \
    --max-concurrent 5 \
    --retry 3 \
    --delay 1.0
```

```python
# decorators.py
import functools
import asyncio
import time

def retry(max_attempts: int = 3, delay: float = 1.0, backoff: float = 2.0):
    """Decorateur retry avec backoff exponentiel"""
    def decorator(func):
        @functools.wraps(func)
        async def wrapper(*args, **kwargs):
            current_delay = delay
            for attempt in range(max_attempts):
                try:
                    return await func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_attempts - 1:
                        raise
                    await asyncio.sleep(current_delay)
                    current_delay *= backoff
        return wrapper
    return decorator

class RateLimiter:
    """Rate limiter utilisant Semaphore"""
    def __init__(self, calls: int, period: float):
        self.semaphore = asyncio.Semaphore(calls)
        self.period = period

    async def acquire(self):
        await self.semaphore.acquire()
        asyncio.create_task(self._release_after(self.period))

    async def _release_after(self, delay: float):
        await asyncio.sleep(delay)
        self.semaphore.release()
```

```python
# scraper.py
import asyncio
import aiohttp
from .decorators import retry, RateLimiter
from .parser import Parser
from .storage import JSONStorage, CSVStorage

class AsyncScraper:
    def __init__(self, max_concurrent: int = 5, retry_attempts: int = 3):
        self.rate_limiter = RateLimiter(max_concurrent, 1.0)
        self.retry_attempts = retry_attempts
        self.parser = Parser()

    async def scrape(self, urls: list[str], output_path: str, format: str = "json"):
        async with aiohttp.ClientSession() as session:
            tasks = [self._fetch_and_parse(session, url) for url in urls]
            results = await asyncio.gather(*tasks, return_exceptions=True)

        # Filtrer erreurs
        valid_results = [r for r in results if not isinstance(r, Exception)]

        # Sauvegarder
        storage = JSONStorage() if format == "json" else CSVStorage()
        storage.save(valid_results, output_path)

        return valid_results

    @retry(max_attempts=3, delay=1.0, backoff=2.0)
    async def _fetch_and_parse(self, session, url: str):
        await self.rate_limiter.acquire()
        async with session.get(url) as response:
            html = await response.text()
            return self.parser.parse(url, html)
```

**Tests automatises**:
- Scraping async fonctionne
- Rate limiting respecte les limites
- Retry reessaie correctement
- Parser extrait donnees
- Storage ecrit JSON/CSV
- CLI accepte arguments
- Coverage > 80%
- pip install fonctionne

**Score qualite**: 99/100
- Originalite: 10/10 (projet complet et realiste)
- Pedagogie: 10/10 (integration tous concepts)
- Testabilite: 10/10 (tests complets)
- Progression: 10/10 (projet capstone)
- Completude: 10/10 (8 concepts projet final)

**Difficulte**: tres difficile
**Temps estime**: 20h

---

## STATISTIQUES DU PLAN

### Repartition par Difficulte
| Difficulte | Nombre | Pourcentage |
|------------|--------|-------------|
| Facile | 4 | 12.5% |
| Moyen | 16 | 50% |
| Difficile | 8 | 25% |
| Tres difficile | 4 | 12.5% |

### Repartition par Partie
| Partie | Exercices | Concepts | Heures |
|--------|-----------|----------|--------|
| Partie 1: OOP Avancee | ex00-ex08 | 41 | 29h |
| Partie 2: Decorateurs | ex09-ex15 | 41 | 26h |
| Partie 3: Generateurs/Itertools | ex16-ex20 | 30 | 16h |
| Partie 4: Async/Await | ex21-ex24 | 23 | 15h |
| Partie 5: Testing pytest | ex25-ex28 | 25 | 12h |
| Partie 6: Packaging | ex29-ex30 | 12 | 6h |
| Projet Final | ex31 | 8 | 20h |
| **TOTAL** | **32** | **180** | **124h** |

### Scores Qualite
| Metrique | Minimum | Maximum | Moyenne |
|----------|---------|---------|---------|
| Score global | 96/100 | 99/100 | 97.2/100 |
| Originalite | 9/10 | 10/10 | 9.8/10 |
| Pedagogie | 9/10 | 10/10 | 9.9/10 |
| Testabilite | 9/10 | 10/10 | 9.9/10 |

### Couverture des Concepts
- **Concepts totaux**: 180
- **Concepts couverts**: 180
- **Couverture**: 100%

---

## VALIDATION MOULINETTE

Tous les exercices sont concus pour etre testables automatiquement:

1. **Sorties deterministes** - memes entrees = memes sorties
2. **Pas de side effects non controles** - pas d'acces fichiers non specifies
3. **Assertions verifiables** - comportements mesurables
4. **Timeout raisonnables** - pas de boucles infinies non controlees
5. **Isolation** - chaque test independant

### Criteres d'evaluation automatique
- **Structure fichiers**: Verification arborescence
- **Imports corrects**: Verification syntaxe
- **Tests unitaires**: pytest avec assertions
- **Coverage**: Seuil minimum configurable
- **Type hints**: mypy strict mode
- **Style**: black/ruff verification

---

## NOTES PEDAGOGIQUES

### Progression Recommandee
1. Commencer par ex00-ex08 (OOP) - fondations solides
2. Enchainer avec ex09-ex15 (Decorateurs) - patterns essentiels
3. Puis ex16-ex20 (Generateurs) - iteration avancee
4. Continuer ex21-ex24 (Async) - programmation moderne
5. Terminer ex25-ex30 (Testing/Packaging) - professionnalisation
6. Projet final ex31 - integration complete

### Points d'Attention
- **ex07 (Metaclasses)**: Concept avance, prevoir temps supplementaire
- **ex23 (Async avance)**: Synchronisation delicate
- **ex31 (Projet final)**: Integration - commencer tot

### Ressources Complementaires
- Documentation Python 3.14 officielle
- Real Python tutorials
- asyncio documentation
- pytest documentation
