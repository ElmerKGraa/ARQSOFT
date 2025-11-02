# Context View

flowchart TB
 subgraph External_Actors["External_Actors"]
        Reader(["Reader"])
        Librarian(["Librarian"])
        Anon(["Anonymous User"])
  end
 subgraph LMS["Library Management System"]
        API(["HTTPS / REST API"])
        AM["Author Management"]
        BM["Book Management"]
        RM["Reader Management"]
        LM["Lending Management"]
        GM["Genre Management"]
  end
    Reader -- authenticate --> Auth["JWT-based Authentication<br>post-registration &amp; login"]
    Librarian -- authenticate --> Auth
    Anon -- register / authenticate --> Auth
    Auth -- token issued --> API
    Reader -- "self-register, lend, return, search" --> API
    Librarian -- manage books, lending ops, reports --> API
    Anon -- browse, register --> API
    API --> AM & BM & RM & LM & GM
    AM --- H2["H2 Database<br>(file-based)"]
    BM --- H2
    RM --- H2
    LM --- H2
    GM --- H2

# Inside book management

```
┌──────────────────────────────────────────────────────────┐
│               bookmanagement                              │
│                                                          │
│  api/                                                    │
│  ┌────────────────────────────────────────────────┐    │
│  │ BookController                                  │    │
│  │ - PUT /api/books/{isbn}                        │    │
│  │ - PATCH /api/books/{isbn}                      │    │
│  │ - GET /api/books/{isbn}                        │    │
│  │ - POST /api/books/search                       │    │
│  │ - GET /api/books/top5                          │    │
│  │ - GET /api/books/suggestions                   │    │
│  │ - GET /api/books/{isbn}/avgDuration            │    │
│  │                                                  │    │
│  │ BookView, BookViewMapper                        │    │
│  └────────────────────────────────────────────────┘    │
│                        ▼                                 │
│  services/                                              │
│  ┌────────────────────────────────────────────────┐    │
│  │ BookService (interface)                         │    │
│  │ BookServiceImpl                                 │    │
│  │ - createBook(CreateBookRequest)                 │    │
│  │ - updateBook(ISBN, UpdateBookRequest)           │    │
│  │ - getBookByIsbn(ISBN)                          │    │
│  │ - searchBooks(Page, Title, Genre, Author)       │    │
│  │ - getTop5Books()                               │    │
│  │ - getSuggestions(ReaderDetails)                │    │
│  │                                                  │    │
│  │ BookMapper (MapStruct)                          │    │
│  └────────────────────────────────────────────────┘    │
│                        ▼                                 │
│  model/                                                 │
│  ┌────────────────────────────────────────────────┐    │
│  │ Book (@Entity)                                  │    │
│  │ - isbn: Isbn (PK)                              │    │
│  │ - title: Title                                  │    │
│  │ - description: Description                      │    │
│  │ - genre: Genre (ManyToOne)                     │    │
│  │ - authors: Set<Author> (ManyToMany)            │    │
│  │ - photo: Photo (OneToOne)                      │    │
│  │                                                  │    │
│  │ Isbn, Title, Description (Value Objects)        │    │
│  └────────────────────────────────────────────────┘    │
│                        ▼                                 │
│  repositories/                                          │
│  ┌────────────────────────────────────────────────┐    │
│  │ BookRepository (interface - domain)             │    │
│  └────────────────────────────────────────────────┘    │
│                        ▼                                 │
│  infrastructure/repositories/impl/                      │
│  ┌────────────────────────────────────────────────┐    │
│  │ SpringDataBookRepository                        │    │
│  │ - extends CrudRepository<Book, Long>            │    │
│  │ - implements BookRepository                     │    │
│  │ - @Query JPQL methods                          │    │
│  │                                                  │    │
│  │ BookRepoCustomImpl                              │    │
│  │ - Criteria API for dynamic searches             │    │
│  └────────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────────┘
```

# INsideModules

```
┌─────────────────────────────────────────────────────────┐
│                    Author (Aggregate Root)               │
├─────────────────────────────────────────────────────────┤
│ - authorNumber: Long (PK, auto-generated)               │
│ - name: Name (VO)                                       │
│ - bio: Bio (VO)                                         │
│ - photo: Photo (Entity)                                 │
│ - version: Long (@Version)                              │
├─────────────────────────────────────────────────────────┤
│ + getBooks(): Set<Book>                                 │
│ + getCoAuthors(): Set<Author>                           │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│                    Book (Aggregate Root)                 │
├─────────────────────────────────────────────────────────┤
│ - pk: Long (internal PK)                                │
│ - isbn: Isbn (VO, unique natural key)                   │
│ - title: Title (VO)                                     │
│ - description: Description (VO)                          │
│ - genre: Genre (ManyToOne, required)                    │
│ - authors: Set<Author> (ManyToMany, min 1)              │
│ - photo: Photo (Entity)                                 │
│ - version: Long (@Version)                              │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│               ReaderDetails (Aggregate Root)             │
├─────────────────────────────────────────────────────────┤
│ - pk: Long (internal PK)                                │
│ - readerNumber: ReaderNumber (VO, YYYY/SEQ)             │
│ - birthDate: BirthDate (VO)                             │
│ - phoneNumber: PhoneNumber (VO)                         │
│ - gdprConsent: boolean (required)                       │
│ - interests: Set<Genre> (ManyToMany)                    │
│ - photo: Photo (Entity)                                 │
│ - user: User (OneToOne)                                 │
│ - version: Long (@Version)                              │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│                  Lending (Aggregate Root)                │
├─────────────────────────────────────────────────────────┤
│ - pk: Long (internal PK)                                │
│ - lendingNumber: LendingNumber (VO, YYYY/SEQ)           │
│ - book: Book (ManyToOne)                                │
│ - readerDetails: ReaderDetails (ManyToOne)              │
│ - startDate: LocalDate                                  │
│ - limitDate: LocalDate (startDate + duration)           │
│ - returnedDate: LocalDate (nullable)                    │
│ - commentary: String                                    │
│ - fineValuePerDayInCents: Integer                       │
│ - version: Long (@Version)                              │
├─────────────────────────────────────────────────────────┤
│ + getFineValue(): int                                   │
│ + getDaysDelayed(): int                                 │
│ + getDaysUntilReturn(): int                             │
│ + getDaysOfLending(): int                               │
│ + setReturned(LocalDate): void                          │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│                    User (Entity)                         │
├─────────────────────────────────────────────────────────┤
│ - pk: Long (internal PK)                                │
│ - username: EmailAddress (VO, unique)                   │
│ - password: Password (VO, BCrypt hashed)                │
│ - name: Name (VO)                                       │
│ - authorities: Set<Role> (READER, LIBRARIAN, ADMIN)     │
│ - enabled: boolean                                      │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│                   Genre (Entity)                         │
├─────────────────────────────────────────────────────────┤
│ - pk: Long (internal PK)                                │
│ - genre: String (unique, max 100 chars)                 │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│                   Photo (Entity)                         │
├─────────────────────────────────────────────────────────┤
│ - pk: Long (internal PK)                                │
│ - photoFile: String (file path)                         │
└─────────────────────────────────────────────────────────┘
```

# Lvl 4

---
config:
  layout: elk
  theme: base
---
classDiagram
    namespace Auth {
        class AuthController {
            -AuthenticationService authService
            +login(LoginRequest) AuthResponse
            +refresh(RefreshTokenRequest) AuthResponse
        }
        class AuthenticationService {
            -UserRepository userRepository
            -JwtTokenUtil jwtTokenUtil
            +authenticate(username, password) AuthResponse
            +refreshToken(token) AuthResponse
        }
        class JwtTokenUtil {
            -PrivateKey privateKey
            -PublicKey publicKey
            +generateToken(User) String
            +validateToken(String) boolean
        }
    }
    namespace UserManagement {
        class UserController {
            -UserService userService
            +create(CreateUserRequest) UserView
            +update(id, UpdateUserRequest) UserView
            +delete(id) void
            +search(SearchRequest) List~UserView~
        }
        class UserService {
            <<interface>>
            +create(CreateUserRequest) User
            +update(id, UpdateUserRequest) User
            +delete(id) void
            +searchUsers(SearchRequest) List~User~
        }
        class UserServiceImpl {
            -UserRepository userRepository
            -PasswordEncoder passwordEncoder
            +create(CreateUserRequest) User
            +update(id, UpdateUserRequest) User
        }
        class User {
            -Long id
            -Long version
            -boolean enabled
            -Username username
            -Password password
            -FullName fullName
            -Set~Role~ authorities
            +isEnabled() boolean
            +getAuthorities() Set~Role~
        }
        class Username {
            <<ValueObject>>
            -String username
            +Username(String)
            +toString() String
        }
        class Password {
            <<ValueObject>>
            -String password
            +Password(String)
            +isEncoded() boolean
        }
        class FullName {
            <<ValueObject>>
            -String fullName
            +FullName(String)
            +toString() String
        }
        class Role {
            <<enumeration>>
            LIBRARIAN
            ADMIN
            READER
        }
        class UserRepository {
            <<interface>>
            +save(User) User
            +findById(Long) Optional~User~
            +findByUsername(Username) Optional~User~
            +searchUsers(Page, SearchRequest) List~User~
        }
        class SpringDataUserRepository {
            -JpaUserRepository jpaRepo
            +save(User) User
            +findById(Long) Optional~User~
        }
    }
    namespace AuthorManagement {
        class AuthorController {
            -AuthorService authorService
            +create(CreateAuthorRequest) AuthorView
            +update(id, UpdateAuthorRequest) AuthorView
            +getAuthorBooks(id) List~BookView~
            +getTop5AuthorsByLending() List~AuthorCountView~
            +getCoAuthors(isbn) List~AuthorView~
        }
        class AuthorService {
            <<interface>>
            +create(CreateAuthorRequest) Author
            +update(id, UpdateAuthorRequest) Author
            +getAuthorBooks(id) List~Book~
            +getTop5() List~AuthorCount~
            +getCoAuthors(isbn) List~Author~
        }
        class AuthorServiceImpl {
            -AuthorRepository authorRepository
            -BookRepository bookRepository
            +create(CreateAuthorRequest) Author
            +update(id, UpdateAuthorRequest) Author
        }
        class Author {
            -Long authorNumber
            -Long version
            -Name name
            -Bio bio
            -List~Book~ books
            +addBook(Book) void
            +removeBook(Book) void
        }
        class Name {
            <<ValueObject>>
            -String name
            +Name(String)
            +toString() String
        }
        class Bio {
            <<ValueObject>>
            -String bio
            +Bio(String)
            +toString() String
        }
        class AuthorRepository {
            <<interface>>
            +save(Author) Author
            +findById(Long) Optional~Author~
            +findTop5ByLending() List~AuthorCount~
            +findCoAuthors(Isbn) List~Author~
        }
        class SpringDataAuthorRepository {
            -JpaAuthorRepository jpaRepo
            +save(Author) Author
            +findTop5ByLending() List~AuthorCount~
        }
    }
    namespace BookManagement {
        class BookController {
            -BookService bookService
            +create(CreateBookRequest) BookView
            +update(isbn, UpdateBookRequest) BookView
            +findByGenre(genre) List~BookView~
            +findByTitle(title) List~BookView~
            +getTop5Books() List~BookCountView~
        }
        class BookService {
            <<interface>>
            +create(CreateBookRequest) Book
            +update(Isbn, UpdateBookRequest) Book
            +findByGenre(String) List~Book~
            +getTop5Books() List~BookCount~
        }
        class BookServiceImpl {
            -BookRepository bookRepository
            -GenreRepository genreRepository
            -AuthorRepository authorRepository
            +create(CreateBookRequest) Book
            +update(Isbn, UpdateBookRequest) Book
        }
        class Book {
            -Long pk
            -Long version
            -Isbn isbn
            -Title title
            -Genre genre
            -Description description
            -List~Author~ authors
            +addAuthor(Author) void
            +removeAuthor(Author) void
        }
        class Isbn {
            <<ValueObject>>
            -String isbn
            +Isbn(String)
            +toString() String
            +isValid() boolean
        }
        class Title {
            <<ValueObject>>
            -String title
            +Title(String)
            +toString() String
        }
        class Description {
            <<ValueObject>>
            -String description
            +Description(String)
            +toString() String
        }
        class BookRepository {
            <<interface>>
            +save(Book) Book
            +findByIsbn(Isbn) Optional~Book~
            +findByGenre(String) List~Book~
            +findTop5ByLendingCount() List~BookCount~
        }
        class SpringDataBookRepository {
            -JpaBookRepository jpaRepo
            +save(Book) Book
            +findByIsbn(Isbn) Optional~Book~
        }
    }
    namespace GenreManagement {
        class GenreController {
            -GenreService genreService
            +create(CreateGenreRequest) GenreView
            +getTop5Genres() List~GenreLendingsView~
            +getAvgLendingDuration() List~GenreLendingsAvgPerMonthView~
        }
        class GenreService {
            <<interface>>
            +create(String) Genre
            +getTop5Genres() List~GenreLending~
            +getAvgDurationPerMonth() List~GenreAvgDto~
        }
        class GenreServiceImpl {
            -GenreRepository genreRepository
            -LendingRepository lendingRepository
            +create(String) Genre
            +getTop5Genres() List~GenreLending~
        }
        class Genre {
            -Long pk
            -String genre
            +getGenre() String
        }
        class GenreRepository {
            <<interface>>
            +save(Genre) Genre
            +findByGenre(String) Optional~Genre~
            +findTop5ByLendingCount() List~GenreLending~
        }
        class SpringDataGenreRepository {
            -JpaGenreRepository jpaRepo
            +save(Genre) Genre
            +findByGenre(String) Optional~Genre~
        }
    }
    namespace ReaderManagement {
        class ReaderController {
            -ReaderService readerService
            +create(CreateReaderRequest) ReaderView
            +update(id, UpdateReaderRequest) ReaderView
            +updatePhoto(id, photo) ReaderView
            +getMonthlyLendings(id) List~ReaderLendingsAvgPerMonthView~
        }
        class ReaderService {
            <<interface>>
            +create(CreateReaderRequest) ReaderDetails
            +update(id, UpdateReaderRequest) ReaderDetails
            +updatePhoto(id, photo) ReaderDetails
            +getMonthlyLendings(id) List~ReaderAvgDto~
        }
        class ReaderServiceImpl {
            -ReaderRepository readerRepository
            -UserRepository userRepository
            -PhotoService photoService
            +create(CreateReaderRequest) ReaderDetails
            +updatePhoto(id, photo) ReaderDetails
        }
        class ReaderDetails {
            -Long pk
            -Long version
            -ReaderNumber readerNumber
            -User user
            -BirthDate birthDate
            -PhoneNumber phoneNumber
            -Photo photo
            -GdprConsent gdprConsent
            -List~Lending~ lendings
            +addLending(Lending) void
            +canBorrow() boolean
        }
        class ReaderNumber {
            <<ValueObject>>
            -String readerNumber
            +ReaderNumber(String)
            +toString() String
        }
        class BirthDate {
            <<ValueObject>>
            -LocalDate birthDate
            +BirthDate(LocalDate)
            +getAge() int
        }
        class PhoneNumber {
            <<ValueObject>>
            -String phoneNumber
            +PhoneNumber(String)
            +toString() String
        }
        class GdprConsent {
            <<ValueObject>>
            -boolean marketing
            -boolean profiling
            -boolean thirdPartySharing
            +hasConsent() boolean
        }
        class ReaderRepository {
            <<interface>>
            +save(ReaderDetails) ReaderDetails
            +findById(Long) Optional~ReaderDetails~
            +findByReaderNumber(ReaderNumber) Optional~ReaderDetails~
            +getMonthlyStats(id) List~ReaderAvgDto~
        }
        class SpringDataReaderRepository {
            -JpaReaderRepository jpaRepo
            +save(ReaderDetails) ReaderDetails
            +findByReaderNumber(ReaderNumber) Optional~ReaderDetails~
        }
    }
    namespace LendingManagement {
        class LendingController {
            -LendingService lendingService
            +create(CreateLendingRequest) LendingView
            +returnBook(id, ReturnBookRequest) LendingView
            +getOverdue() List~LendingView~
            +getAvgPerMonth() List~LendingView~
        }
        class LendingService {
            <<interface>>
            +create(CreateLendingRequest) Lending
            +returnBook(id, commentary) Lending
            +getOverdue() List~Lending~
            +getAvgDurationPerMonth() List~LendingAvgDto~
        }
        class LendingServiceImpl {
            -LendingRepository lendingRepository
            -ReaderRepository readerRepository
            -BookRepository bookRepository
            +create(CreateLendingRequest) Lending
            +returnBook(id, commentary) Lending
        }
        class Lending {
            -Long pk
            -Long version
            -ReaderDetails reader
            -Book book
            -StartDate startDate
            -DueDate dueDate
            -ReturnedDate returnedDate
            -Commentary commentary
            -Fine fine
            +isOverdue() boolean
            +returnBook(date, commentary) void
            +getDaysOverdue() int
        }
        class StartDate {
            <<ValueObject>>
            -LocalDate startDate
            +StartDate(LocalDate)
            +toLocalDate() LocalDate
        }
        class DueDate {
            <<ValueObject>>
            -LocalDate dueDate
            +DueDate(LocalDate)
            +isOverdue() boolean
        }
        class ReturnedDate {
            <<ValueObject>>
            -LocalDate returnedDate
            +ReturnedDate(LocalDate)
            +toLocalDate() LocalDate
        }
        class Commentary {
            <<ValueObject>>
            -String commentary
            +Commentary(String)
            +toString() String
        }
        class Fine {
            -Long pk
            -Lending lending
            -FineAmount fineAmount
            -boolean settled
            +settle() void
            +getAmount() FineAmount
        }
        class FineAmount {
            <<ValueObject>>
            -BigDecimal amount
            +FineAmount(BigDecimal)
            +add(FineAmount) FineAmount
        }
        class LendingRepository {
            <<interface>>
            +save(Lending) Lending
            +findById(Long) Optional~Lending~
            +findOverdue() List~Lending~
            +getAvgDurationPerMonth() List~LendingAvgDto~
        }
        class SpringDataLendingRepository {
            -JpaLendingRepository jpaRepo
            +save(Lending) Lending
            +findOverdue() List~Lending~
        }
    }
    User "1" *-- "1" Username : contains
    User "1" *-- "1" Password : contains
    User "1" *-- "1" FullName : contains
    User "1" -- "0..*" Role : has
    UserServiceImpl ..> UserRepository : uses
    UserController ..> UserService : uses
    SpringDataUserRepository ..|> UserRepository : implements
    UserServiceImpl ..|> UserService : implements
    AuthController ..> AuthenticationService : uses
    AuthenticationService ..> UserRepository : uses
    AuthenticationService ..> JwtTokenUtil : uses
    Author "1" *-- "1" Name : contains
    Author "1" *-- "0..1" Bio : contains
    Author "1" o-- "0..*" Book : writes
    AuthorServiceImpl ..> AuthorRepository : uses
    AuthorServiceImpl ..> BookRepository : uses
    AuthorController ..> AuthorService : uses
    SpringDataAuthorRepository ..|> AuthorRepository : implements
    AuthorServiceImpl ..|> AuthorService : implements
    Book "1" *-- "1" Isbn : contains
    Book "1" *-- "1" Title : contains
    Book "1" *-- "0..1" Description : contains
    Book "1" --> "1" Genre : categorized by
    Book "0..*" o-- "1..*" Author : written by
    BookServiceImpl ..> BookRepository : uses
    BookServiceImpl ..> GenreRepository : uses
    BookServiceImpl ..> AuthorRepository : uses
    BookController ..> BookService : uses
    SpringDataBookRepository ..|> BookRepository : implements
    BookServiceImpl ..|> BookService : implements
    GenreServiceImpl ..> GenreRepository : uses
    GenreServiceImpl ..> LendingRepository : uses
    GenreController ..> GenreService : uses
    SpringDataGenreRepository ..|> GenreRepository : implements
    GenreServiceImpl ..|> GenreService : implements
    ReaderDetails --|> User : extends
    ReaderDetails "1" *-- "1" ReaderNumber : contains
    ReaderDetails "1" *-- "1" BirthDate : contains
    ReaderDetails "1" *-- "0..1" PhoneNumber : contains
    ReaderDetails "1" *-- "1" GdprConsent : contains
    ReaderDetails "1" o-- "0..*" Lending : has
    ReaderServiceImpl ..> ReaderRepository : uses
    ReaderServiceImpl ..> UserRepository : uses
    ReaderController ..> ReaderService : uses
    SpringDataReaderRepository ..|> ReaderRepository : implements
    ReaderServiceImpl ..|> ReaderService : implements
    Lending "1" *-- "1" StartDate : contains
    Lending "1" *-- "1" DueDate : contains
    Lending "1" *-- "0..1" ReturnedDate : contains
    Lending "1" *-- "0..1" Commentary : contains
    Lending "1" --> "1" ReaderDetails : borrowed by
    Lending "1" --> "1" Book : borrows
    Lending "0..1" o-- "0..1" Fine : may have
    Fine "1" *-- "1" FineAmount : contains
    LendingServiceImpl ..> LendingRepository : uses
    LendingServiceImpl ..> ReaderRepository : uses
    LendingServiceImpl ..> BookRepository : uses
    LendingController ..> LendingService : uses
    SpringDataLendingRepository ..|> LendingRepository : implements
    LendingServiceImpl ..|> LendingService : implements



