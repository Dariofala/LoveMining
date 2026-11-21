# 💘 LoveMining — Dating Platform on MongoDB and Neo4j

![Java](https://img.shields.io/badge/Java-21-ED8B00?logo=openjdk&logoColor=white)
![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.4-6DB33F?logo=springboot&logoColor=white)
![MongoDB](https://img.shields.io/badge/MongoDB-replica%20set-47A248?logo=mongodb&logoColor=white)
![Neo4j](https://img.shields.io/badge/Neo4j-graph-4581C3?logo=neo4j&logoColor=white)
![Swagger](https://img.shields.io/badge/OpenAPI-Swagger%20UI-85EA2D?logo=swagger&logoColor=black)

LoveMining is a dating platform that matches people by **geographic proximity** and **shared interests**.
It combines a **document database (MongoDB)** for profiles, reviews and analytics with a **graph database (Neo4j)**
for likes, matches and recommendations, coordinated by a **Spring Boot REST API**.

Project for the **Large-Scale and Multi-Structured Databases** course — MSc in Artificial Intelligence and
Data Engineering, University of Pisa, academic year 2025/26.

---

## ✨ Highlights

- **Polyglot persistence**: each database does what it is best at — MongoDB for rich user documents and aggregations,
  Neo4j for relationships and graph traversals.
- **Distributed MongoDB**: 3-node replica set with election priorities, `nearest` read preference and a throughput-oriented
  write concern (`w:1, j:false`); sharding strategy analysed for future growth (city + hashed user ID).
- **Cross-database consistency** handled in the application layer for registration, profile updates and deletions,
  with compensating actions when one of the two writes fails.
- **Graph-based recommendations**: ten compatible partners filtered by city/state, age range and orientation,
  ranked by number of shared interests.
- **Admin analytics** with MongoDB aggregation pipelines and Cypher queries.
- **Secure REST API**: Spring Security with HTTP Basic auth, BCrypt-hashed passwords, user/admin roles, Swagger UI.
- **Large dataset**: ~60,000 real anonymised profiles (OkCupid, Kaggle) plus a synthetic interaction layer of
  **1.1 million likes/dislikes**, ~3,500 matches and as many reviews; indexes evaluated with before/after benchmarks.

## 🧩 Data model

**MongoDB**
- `users` — profile, demographics, extracted interests and an embedded summary of the reviews made by the user.
- `reviews` — rating, comment, author, target and date.

**Neo4j**
```
(:User)-[:LIVES_IN]->(:City)-[:LOCATED_IN]->(:State)
(:User)-[:HAS_INTEREST]->(:Interest)
(:User)-[:LIKES]->(:User)        (:User)-[:DISLIKES]->(:User)
```
A match is a pair of reciprocal `LIKES`. Interests are extracted from the free-text essays of each profile.

## 🔌 REST API

| Area | Method | Endpoint | Description |
|------|--------|----------|-------------|
| Auth | `POST` | `/api/authentication/register` | Register a new user |
| User | `GET` | `/api/users` | Profile of the authenticated user |
| User | `PATCH` | `/api/users/me` | Update the profile |
| User | `GET` | `/api/users/{id}` | View a user |
| User | `POST` | `/api/users/{id}/like` · `/dislike` | Like / dislike a user |
| User | `GET` | `/api/users/matches` | Your matches |
| User | `GET` | `/api/users/recommendations/{filters}?locationFilter=City\|State&minAge=&maxAge=` | Personalised recommendations |
| User | `POST` | `/api/users/{id}/review` | Review a user you matched with |
| User | `GET` | `/api/users/reviews` | Reviews you wrote |
| Admin | `GET` | `/api/admin/user/{id}` · `/api/admin/userReviews/{id}` | Look up users and reviews |
| Admin | `DELETE` | `/api/admin/users/{id}` | Delete a user from both databases |

### Analytics (admin)

| Endpoint | Database | What it computes |
|----------|----------|------------------|
| `/api/admin/analytics/love-points?state=` | Neo4j | "Love Points" score of each city in a state, weighting mutual matches more than likes |
| `/api/admin/analytics/unhappy-cities` | MongoDB | Top 5 cities with the most negative reviewers (extreme vs moderate) |
| `/api/admin/analytics/glow-up` | MongoDB | Users whose ratings improved the most in the last six months |
| `/api/admin/analytics/status-by-age-group` | MongoDB | Share of singles per age group |
| `/api/admin/analytics/orientation-by-age-group` | MongoDB | Orientation distribution with nested age breakdown |

## 🗂️ Repository structure

```
├── LoveMining/                    # Spring Boot application (Maven)
│   └── src/main/java/it/unipi/LoveMining/
│       ├── config/                # MongoDB, security, OpenAPI configuration
│       ├── controller/            # REST controllers (auth, user, admin)
│       ├── model/{mongo,neo4j}/   # documents and graph nodes
│       ├── repository/            # Spring Data repositories and custom queries
│       └── service/               # business logic and cross-database consistency
├── Dataset/                       # ready-to-import datasets + generation scripts (see Dataset/README.md)
└── Documentation_LoveMining.pdf   # full report: design, data models, queries, indexes
```

## 🚀 Running it

**Requirements:** Java 21, MongoDB, Neo4j 5.

1. Import the datasets from [`Dataset/`](Dataset/) into MongoDB and Neo4j, following [`Dataset/README.md`](Dataset/README.md).
2. Configure the connections through environment variables (defaults point to `localhost`):

   ```bash
   export MONGODB_URI="mongodb://localhost:27017"
   export NEO4J_URI="bolt://localhost:7687"
   export NEO4J_USERNAME="neo4j"
   export NEO4J_PASSWORD="your-password"
   ```

3. Start the application:

   ```bash
   cd LoveMining
   ./mvnw spring-boot:run
   ```

4. Open Swagger UI at `http://localhost:8080/swagger-ui.html` and log in with one of the test accounts listed in
   [`Dataset/README.md`](Dataset/README.md).

## ⚠️ Notes

- User profiles come from the public, anonymised [OkCupid Profiles dataset](https://www.kaggle.com/datasets/andrewmvd/okcupid-profiles);
  emails, passwords, likes, matches and reviews are synthetic.
- Consistency between MongoDB and Neo4j is managed by the application, not by distributed transactions.
- The project was developed and tested on a university cluster; the repository has been adapted to run locally.

## 👥 Authors

- **Andrea Doni** — [@Andryd22](https://github.com/Andryd22)
- **Dario Falaschi** — [@Dariofala](https://github.com/Dariofala)
- **Leonardo Lentini** — [@LeonardoLentini](https://github.com/LeonardoLentini)
