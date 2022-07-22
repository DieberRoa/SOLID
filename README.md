# Principios Solid con Swift Playgrounds

![Swift Solid](https://drive.google.com/uc?id=1vLdrH2QFX1vWxzJ0biShaGpo82nph_f9)

---
## Introducción

Para los ejemplos de implementación de los principios [SOLID](https://www.freecodecamp.org/news/solid-principles-explained-in-plain-english/) en Swift se van a realizar las clases pertinentes a un juego de naves que puedan disparar , con enemigos y un jugador, además de una app sencilla que lista nombres con SwiftUI.

[SOLID](https://en.wikipedia.org/wiki/SOLID) es el acrónimo de los siguientes principios:

**S**ingle responsability <br>
**O**pen/closed <br>
**L**iskov sustitution<br> 
**I**nterface segregation<br>
**D**ependency inversion<br>

---

## Principio de única responsabilidad
Se empieza construyendo una nave así: 

![Ship Model](https://drive.google.com/uc?id=1MXndJeethxHBTIg0SOjvjt0BlDUbJZXV )


``` Swift
class Ship {
    
    var lives = 3
    var x = 0
    var y = 0
    
    
    func dye() {
        lives = lives - 1
    }
    
    func shot() {
        // Shot
    }
    
    func move(toX: Int, toY: Int) {
        // moves from actual x,y to a toX, toY
    }
}
```
Esta clase tiene mucha responsabilidades: como morir, disparar y moverse, es necesario segregar estas gestión a diversas clases: una se encarga de la muerte, otra para que gestione el disparo y otra para que gestione el movimiento, asi:

![Ship Model 2](https://drive.google.com/uc?id=1P2JaoHEboC-YNyFo5yNFN3XYISJARC81 )

``` Swift
// Manejador de las vidas
class LiveHandler {
    
    var lives = 3
    
    init(lives : Int) {
        self.lives = lives
    }
    
    func dye() {
        lives = lives - 1
    }
}

// Manejador de los disparos
class ShotHandler {
    
    func shot() {
        // Shot
    }
    
}

// Coordenadas 
struct Position {
    
    var x = 0
    var y = 0
    
    init (_ x: Int,_ y: Int) {
        self.x = x
        self.y = y
    }
    
}

// Manejador del movimiento 
class MoveHandler {
    
    var position: Position
    
    init(x: Int, y: Int) {
        self.position = Position(x,y)
    }
    
    func move(toX: Int, toY: Int) {
        // do move 
        // set the last position
        position.x = toX
        position.y = toY
    }
}

// 
class BestShip {
    
    let liveHandler: LiveHandler
    let shotHandler : ShotHandler
    let moveHandler : MoveHandler
    
    init (liveHandler: LiveHandler, shotHandler : ShotHandler, moveHandler : MoveHandler) {
        self.liveHandler = liveHandler
        self.shotHandler = shotHandler
        self.moveHandler = moveHandler
    }
    
    func getLives() -> Int {
        return self.liveHandler.lives
    }
    
    func getPosition() -> Position {
        return self.moveHandler.position
    }
    
    func dye() {
        liveHandler.dye()
    }
    
    func shot() {
        shotHandler.shot()
    }
    
    func move(toX: Int, toY: Int) {
        moveHandler.move(toX: toX, toY: toY)
    }
    
}
```
Aunque se tienen los métodos estos no contienen la lógica asociada.

---

## Principio Abierto / Cerrado

Si se va a ampliar la nave para que se pueda estrellar no es necesario modificar el código de la clase BestShip se puede extender así con CollisionableShip:

![Ship Model 3](https://drive.google.com/uc?id=1rxO1mHb0akbwK8FR8GoUE6tdokVPGrMU )

``` Swift
class CollisionableShip : BestShip {
    
    var isCollision : Bool = false
    
    func collision() -> Bool {
        // Crash with other object code
        return isCollision
    }
    
}
```

Aplicando Single responsability:

![Ship Model 4](https://drive.google.com/uc?id=1RoYVZZu9guMsyD3tI5ITZ8Ba2Ih5Wa98 )

``` Swift
// Manejador de colisiones
class CollisionHandler {
    
    var shipsInScene : [BestShip] = []
    var ship : BestShip 
    var isCollision : Bool = false
    
    init(ship: BestShip) {
        self.ship = ship
    }
    
    func collision() -> Bool {
        // collision logic
        return isCollision
    }
    
}

// Implementación final
class BestCollisionableShip : BestShip {
    
    var collisionHandler : CollisionHandler
    
    init(liveHandler: LiveHandler, shotHandler : ShotHandler, moveHandler : MoveHandler, collisionHandler: CollisionHandler) {
        self.collisionHandler = collisionHandler
        super.init(liveHandler: LiveHandler>, shotHandler: ShotHandler, moveHandler: MoveHandler)
    }
    
    func collision() -> Bool {
        return collisionHandler.collision()
    }
}
```
---

## Principio de sustitución de Liskov

Para este ejemplo se crearán varios hijos de BigShip, el principio nos da las reglas para construir buenas herencias entre clases aunque en la práctica es díficil de ver la violación en código. 

Los hijos se pueden usar como su padre sin la necesidad de conocer la diferencia entre ellas.

Se crean las clases de Ship y Bullet que heredan de FlyObject y Player y Enemy que heredan de Ship, así:


![Ship Model 5](https://drive.google.com/uc?id=10HNAmgE4zQv_HiLz4c-bhueexWtdpPL0 )

Ver en la página **SOLID3** del Playground él código completo, por ahora se mira la clase Game: 

``` Swift
class Game {
    
    var player : Player?
    var enemies : [Enemy] = []
    var characters : [FlyObject] = []
    
    init() {        
        self.player = Player(moveHandler: MoveHandler(x: 10, y: 10), shotHandler: ShotHandler())
        self.player?.collisionHandler = CollisionHandler(ship: self.player!)
        self.characters.append(self.player!)
        createEnemies()
    }
    
    func createEnemies() {
        for enemiesCount in 1 ... 20 {
            var newEnemy = Enemy(moveHandler: MoveHandler(x: 10, y: 10), shotHandler: ShotHandler())
            newEnemy.collisionHandler = CollisionHandler(ship: newEnemy)
            for bulletCoung in 1...2 {
                self.characters.append(newEnemy.shot())
            }
            self.characters.append(newEnemy)
            self.enemies.append(newEnemy)
        }
    }
    
    func showCharactersCount() -> Int {
        return self.characters.count
    }
    
    func showCharactersLives() -> [Int] {
        var lives : [Int] = []
        self.characters.forEach { character in
            lives.append(character.liveHandler.lives)
        }
        return lives
    }
    
    func showCharactersVelocity() -> [Int] {
        // violates principle
        var velocities : [Int] = []
        self.characters.forEach { character in
            velocities.append(character.velocity)
        }
        return velocities
    }
    
}

let game = Game()
print(game.showCharactersCount())
print(game.showCharactersLives())
```

En el atributo characters de tipo ```[FlyObject]``` podemos agregar objetos ```Enemy```,```Bullet``` y ```Player``` se puede hacer porque ```Enemy```, ```Bulley``` y ```Player``` son hijos de ```FlyObject```. En el método ```showCharactersVelocity()``` se rompe el código debido a que algunos ```FlyObject``` no tienen el atributo ```velocity ``` (encontrado en ```Bullet```) para arreglar esto es necesario discrimar el tipo en este caso en la utilización, pero, también podría subirlo a una clase superior. Entre más discriminaciones de este tipo peor es la herencia.

---

## Segregación de Interfaces

Este principio enseña que ninguna clase debe depender de métodos que no usa, es por eso que se realizan contratos para que cada clase pueda implementarlos.

En esta implementación: 

![Ship Model 6](https://drive.google.com/uc?id=1O5oHCK9oYgxpGIhoeKawFyS0wOi46WKU )

Se le da a la clase ```Ship``` el comportamiento ```Shoterable``` para que pueda disparar y a la clase ```Bullet``` el comportamiento ```Velocitable``` para que pueda manejar la velocidad, la idea es que aunque ```Bullet``` y ```Ship``` sean hijas de ```FlyObject``` puedan tener comportamientos diferentes si ```FlyObject``` tuviese ```Shoterable``` y ```Velocitable``` la clase ```Ship``` tendría una implementación vacía o inútil del método ```setDirectionVelocity(direction: double, velocity: Int)``` al igual que ```Bullet``` tendría un implementación vacía o inútil del método ```shot()```.

Ver en la página **SOLID4** del Playground.

---

## Principio de inversión de dependencias

Este principio nos enseña que un modulo no debe depender de otro modulo sino de abstracciones y las abstracciones no deben depender de los detalles.

En este ejemplo tenemos una aplicación tipica en capas, en este caso la capa superior conoce los detalles de la capa anterior, además la implementación se encuentra fácilmente y cada capa depende del detalle de la misma.

![Layer Model 1](https://drive.google.com/uc?id=1Q35ZCX79eqBt0JjS1YSXOFqOD5qhVvTm )

```PlayersManagement``` depende de la instancia de ```PlayerRepository```, así como la capa de presentación depende de la instacia directa de la lógica del negocio. 

Ver en la página **SOLID5.1** del Playground.

``` Swift
// transversal
struct Model {
    struct Player {
        var id = UUID() 
        var nickName : String
    }
}

// Data Layer
struct Data {
    
    class PlayerRepository {
        
        func get() -> [Model.Player] {
            return getPlayers()
        }
        
        private func getPlayers() -> [Model.Player] {
            var players : [Model.Player] = []
            players.append(Model.Player(nickName: "CoolMan"))
            players.append(Model.Player(nickName: "SeriouslyStuff1022"))
            players.append(Model.Player(nickName: "ThornSkillex45"))
            players.append(Model.Player(nickName: "SrPensionao"))
            players.append(Model.Player(nickName: "DaromDalashim"))
            players.append(Model.Player(nickName: "ItzWatEver90"))
            players.append(Model.Player(nickName: "ZMilof332"))
            players.append(Model.Player(nickName: "Esperpento43531"))
            players.append(Model.Player(nickName: "HereYourDead102"))
            players.append(Model.Player(nickName: "AnotherGuyProgrammer"))
            return players
        }
    }
    
}

// Bussiness Layer
struct BussinessLogic {
    
    class PlayersManagement {
        
        func getPlayers() -> [Model.Player] {
            return sort()
        }
        
        private func sort() -> [Model.Player] {
            var players = Data.PlayerRepository().get()
            players.sort {
                $0.nickName < $1.nickName
            }
            return players
        }
        
    }
    
}

// Presentation Layer
import SwiftUI

struct Presentation {
    
    struct PlayersView: View {
        
        let players : [Model.Player] = BussinessLogic.PlayersManagement().getPlayers()
        
        var body: some View {
            VStack {
                List {
                    ForEach(players, id: \.id) { player in
                        HStack {
                            Image(systemName: "star.circle").foregroundColor(.green)
                            VStack {
                                Text(player.nickName).bold()
                            }
                        }
                    }
                }
            }
        }
        
    }
}

// Ejecucion

import PlaygroundSupport
PlaygroundPage.current.setLiveView(Presentation.PlayersView())
```
Para evitar esto la comunicación entre capas debe depender de abstracciones y no de los detalles, es decir con contratos , así:

![Layer Model 2](https://drive.google.com/uc?id=1-c6UxYjXRg48WKcNsnDhUDB2Khvyr7tj)

La capa de ```BussinessLogic``` depende del contrato ```IPlayersRepository``` y no de la instancia de ```PlayerRepository``` al igual que la capa de presentación depende del contrato establecido para la lógica del negocio.

Ver en la página **SOLID5.2** del Playground.

```Swift
// transversal
struct Model {
    struct Player {
        var id = UUID() 
        var nickName : String
    }
}


// Data Layer

protocol IPlayerRepository {
    func get() -> [Model.Player]
}

struct Data {
    
    class PlayerRepository: IPlayerRepository {
        
        func get() -> [Model.Player] {
            return getPlayers()
        }
        
        private func getPlayers() -> [Model.Player] {
            var players : [Model.Player] = []
            players.append(Model.Player(nickName: "CoolMan"))
            players.append(Model.Player(nickName: "SeriouslyStuff1022"))
            players.append(Model.Player(nickName: "ThornSkillex45"))
            players.append(Model.Player(nickName: "SrPensionao"))
            players.append(Model.Player(nickName: "DaromDalashim"))
            players.append(Model.Player(nickName: "ItzWatEver90"))
            players.append(Model.Player(nickName: "ZMilof332"))
            players.append(Model.Player(nickName: "Esperpento43531"))
            players.append(Model.Player(nickName: "HereYourDead102"))
            players.append(Model.Player(nickName: "AnotherGuyProgrammer"))
            return players
        }
    }
    
}

// Bussiness Layer

protocol IPlayersManagement {
    func getPlayers() -> [Model.Player]
}

struct BussinessLogic {
    
    class PlayersManagement: IPlayersManagement {
        
        private var repository : IPlayerRepository
        
        init(repository: IPlayerRepository) {
            self.repository = repository
        }
        
        func getPlayers() -> [Model.Player] {
            return sort()
        }
        
        private func sort() -> [Model.Player] {
            var players = repository.get()
            players.sort {
                $0.nickName < $1.nickName
            }
            return players
        }
        
    }
    
}

// Presentation Layer
import SwiftUI

struct Presentation {
    
    struct PlayersView: View {
        
        private var playersManagement : IPlayersManagement
        private var players : [Model.Player] = []
        
        init(playersManagement: IPlayersManagement) {
            self.playersManagement = playersManagement
            players = self.playersManagement.getPlayers()
        }
        
        var body: some View {
            VStack {
                List {
                    ForEach(players, id: \.id) { player in
                        HStack {
                            Image(systemName: "star.circle").foregroundColor(.green)
                            VStack {
                                Text(player.nickName).bold()
                            }
                        }
                    }
                }
            }
        }
        
    }
}

// Ejecucion

import PlaygroundSupport

let view = Presentation.PlayersView(
    playersManagement: BussinessLogic.PlayersManagement(
        repository: Data.PlayerRepository()))

PlaygroundPage.current.setLiveView(view)
```

## Conclusión

Es importante conocer los principios SOLID para poder continuar con las implementaciones de los patrones necesarios para promover buena codificación, mantenibilidad y modularidad.

En estas fuentes se encuentra el código y el texto:

- GitLab Pragma: [Chapter Mobile - Solid](https://gitlab.com/chapter-mobile/comunidad-ios/playgrounds/solid) (hay que pedir acceso, también se puede hacer un fork)
- [Mi GitHub Personal](https://github.com/DieberRoa/SOLID)

Este texto también se encuentra en: [Alejandría Pragma](https://alejandria.pragma.com.co/books/principios-solid-con-swift-playgrounds)

Cualquier sugerencia se puede hacer en los canales del Chapter:

- [Workplace](https://pragma.workplace.com/groups/pragma.movilidad)
- [Foros](https://comunidad.pragma.com.co/t/principios-solid-con-swift-playground/918)

