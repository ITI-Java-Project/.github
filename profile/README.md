# System Main Components

```mermaid
graph TB

subgraph Client["Client Side"]
    direction TB

    UI["Landing / Login / Start / Game / Recordings UI"]
    ConnectionManager
    ConnectionStrategy
    LocalConnection
    OnlineConnection
    ClientMessageListener
    Authentication
    UserSession
    SessionDAO
    GameStrategy
    LocalGame
    AiGame
    OnlineGame
    CPUStrategy
    MiniMax
    Board
    MatchRecorder
    MatchFileHandler
    RecordingUI["RecordingController / RecordingCardController / RecordedMatchController"]
    DTOsClient["Player DTO / Session DTO / Match DTO"]

    UI --> ConnectionManager
    UI --> RecordingUI
    UI --> DTOsClient

    Authentication --> ConnectionManager

    ConnectionManager --> ConnectionStrategy
    ConnectionStrategy --> LocalConnection
    ConnectionStrategy --> OnlineConnection
    ConnectionStrategy --> ClientMessageListener

    ClientMessageListener --> SessionDAO
    ClientMessageListener --> UserSession

    GameStrategy --> MatchRecorder
    MatchRecorder --> MatchFileHandler

    GameStrategy --> LocalGame
    GameStrategy --> AiGame
    GameStrategy --> OnlineGame

    AiGame --> CPUStrategy
    CPUStrategy --> MiniMax

    LocalGame --> Board
    AiGame --> Board
    OnlineGame --> Board
end

subgraph Network
    Socket["TCP Socket (Port 5000)"]
end

subgraph Server["Server Side"]
    direction TB

    ServerService
    ServerListener
    ClientHandler
    ServerController
    SessionManager
    Session
    DbManager
    PlayerDao
    SessionDao
    DTOsServer["PlayerDto / SessionDto"]
    Database["Derby DB"]

    ServerService --> ServerListener
    ServerListener --> SessionManager
    SessionManager --> Session

    ServerService --> ClientHandler
    ClientHandler --> ServerListener

    Session --> PlayerDao
    Session --> SessionDao

    PlayerDao --> DbManager
    SessionDao --> DbManager
    DbManager --> Database

    ServerController --> ServerService
    ServerListener --> DTOsServer
end

Client --> Socket
Socket --> ServerService
```

# Database Schema

```mermaid
erDiagram
    PLAYER {
        int ID PK
        string NAME
        string EMAIL
        string PASSWORD
        string GENDER
        int SCORE
    }

    SESSION {
        int ID PK
        int PLAYER1_ID FK
        int PLAYER2_ID FK
        string PLAYER1_NAME
        string PLAYER2_NAME
        int PLAYER1_SCORE
        int PLAYER2_SCORE
        datetime START_TIME
        datetime END_TIME
        int WINNER_ID FK
    }

    PLAYER ||--o{ SESSION : "player1"
    PLAYER ||--o{ SESSION : "player2"
    PLAYER ||--o{ SESSION : "winner"
```

# UML Class Diagrams

## Client Side

```mermaid
classDiagram
    class ConnectionStrategy {
        <<interface>>
        +boolean connectToServer()
        +void startListener(GameCallback)
        +void send(String)
    }

    class LocalConnection {
        +InetAddress ip
        +int portNumber
        +boolean connectToServer()
    }
    
    class OnlineConnection {
        +InetAddress ip
        +int portNumber
        +boolean connectToServer()
    }

    class ConnectionManager {
        -ConnectionStrategy currentConnection
        +init(ConnectionStrategy)
        +getConnection() ConnectionStrategy
    }

    class ClientMessageListener {
        -ConnectionStrategy conn
        -Callback callback
        +run()
        -List~Player~ getAllPlayers(String response)
    }

    class UserSession {
        <<Singleton>>
        -Player currentUser
        +getInstance()
        +setCurrentUser(Player)
        +getCurrentUser()
    }

    class SessionDAO {
        <<Singleton>>
        -Session currentSession
        +setSessionFromJson(String)
        +getSession() Session
        +hasActiveSession() boolean
        +updatePlayer1Score(int)
        +updatePlayer2Score(int)
        +clearSession()
    }

    class GameStrategy {
        <<interface>>
        +void play(int row,int col,char)
        +boolean win()
        +void turn()
    }

    class LocalGame {
        +void play(int row,int col,char)
    }

    class AiGame {
        -CPUStrategy cpu
        +void play(int row,int col,char)
        +void playAi(char)
    }

    class OnlineGame {
        -ConnectionStrategy connection
        +void play(int row,int col,char)
        +void receivePlay(int row,int col)
    }

    class CPUStrategy {
        <<interface>>
        +int[] decideMove(Board board)
    }

    class MiniMax {
        +int[] decideMove(Board board)
        +int[] randomMove(Board board)
    }

    class Board {
        <<Singleton>>
        -char[][] board
        +reset()
        +getBoard() char[][]
        +getCell(int row,int col) char
    }

    class MatchRecorder {
        -List~Move~ moves
        -MatchFileHandler fileHandler
        +addMove(Move)
        +saveRecord()
        +loadRecord()
    }

    class MatchFileHandler {
        +save(Match)
        +load(String) Match
    }

    class RecordingController {
        -VBox recordsContainer
        +loadRecord(String playerName,String date,String time,MatchResult result,Match match)
    }

    class RecordingCardController {
        -HBox cardRoot
        -ImageView playerImage
        -Label vsLabel
        -Label dateLabel
        -Label timeLabel
        +setData(String,String,String,MatchResult)
    }

    class RecordedMatchController {
        -Move[] moves
        -int currentMoveIndex
        +displayMatchInfo()
        +clearBoard()
        +playReplay()
    }

    class Player {
        -int id
        -int score
        -String name
        -String email
        -String gender
        +getId()
        +getName()
    }

    class Session {
        -int id
        -int player1Id
        -int player2Id
        -int player1Score
        -int player2Score
        -String player1Name
        -String player2Name
        +getId()
        +getPlayer1Id()
        +getPlayer2Id()
    }

    class Move {
        -int row
        -int col
        -char symbol
        +getRow()
        +getCol()
        +getSymbol()
    }

    class Match {
        -String oponentName
        -Date date
        -Move[] moves
        -MatchResult result
        +getOponentName()
        +getDate()
    }

    class MatchResult {
        <<enumeration>>
        WIN
        LOSE
        DRAW
    }

    ConnectionStrategy <|.. LocalConnection
    ConnectionStrategy <|.. OnlineConnection
    GameStrategy <|.. LocalGame
    GameStrategy <|.. AiGame
    GameStrategy <|.. OnlineGame
    CPUStrategy <|.. MiniMax
    AiGame --> MiniMax
    MatchRecorder --> MatchFileHandler
    RecordingController --> RecordingCardController
    RecordingController --> RecordedMatchController
```

## Server Side

```mermaid
classDiagram
    class DbManager {
        <<Singleton>>
        +ResultSet getQuery(String)
        +int insertAndGetId(String, Object...)
        +boolean updateQueryPrepared(String, Object...)
    }

    class PlayerDao {
        +PlayerDto register(String name,String email,String password)
        +PlayerDto login(String username,String password)
        +List~PlayerDto~ getAllPlayers()
        +boolean increaseWinnerScore(int playerId)
    }

    class SessionDao {
        +SessionDto createSession(int player1Id,int player2Id,String player1Name,String player2Name)
        +boolean updateSessionScores(int sessionId,int player1Score,int player2Score)
        +SessionDto getSessionById(int id)
    }

    class PlayerDto {
        -int id
        -String name
        -String email
        -String password
        -int score
        -String gender
        +getId()
        +getName()
    }

    class SessionDto {
        -int id
        -int player1Id
        -int player2Id
        -String player1Name
        -String player2Name
        -int player1Score
        -int player2Score
        +getId()
        +getPlayer1Id()
        +getPlayer2Id()
        +setPlayer1Score(int)
        +setPlayer2Score(int)
    }

    class ServerListener {
        -SessionManager sessionManager
        +onMessage(String msg, ClientHandler client)
        -handleInvitationRequest(String msg,ClientHandler)
        -sendSessionDataAsJson(SessionDto,ClientHandler,ClientHandler)
    }

    class ClientHandler {
        -Socket socket
        -PlayerDto player
        +send(String)
        +getPlayer()
    }

    class SessionManager {
        -Queue~ClientHandler~ waitingClients
        -List~Session~ activeSessions
        +addPlayer(ClientHandler)
        +createSession(ClientHandler,ClientHandler) SessionDto
        +getAvailablePlayersAsJsonString(ClientHandler)
        +finishSession(Session)
        +getWaitingClients()
        +getPlayersInSessionCount()
    }

    class Session {
        -ClientHandler x
        -ClientHandler o
        -char turn
        -int sessionId
        -int player1Score
        -int player2Score
        +playMove(ClientHandler,int,int)
        +broadcast(String)
        +endSession()
    }

    class ServerService {
        -int port
        -ServerListener listener
        +start()
        +shutdown()
    }

    class ServerController {
        +startServer()
        +stopServer()
        +updateBarChartData()
    }

    DbManager <.. PlayerDao
    DbManager <.. SessionDao
    ServerListener --> SessionManager
    SessionManager --> Session
    Session --> PlayerDao
    SessionDao --> DbManager
    ClientHandler --> ServerListener
    ServerService --> ServerListener
    ServerController --> ServerService
```
