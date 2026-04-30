Separa las operaciones de mutación de estado (Commands) de las operaciones de lectura (Queries).
* **Commands:** Modelos y DBs optimizados para consistencia transaccional (ej. SQL).
* **Queries:** Modelos y DBs optimizados para alto rendimiento de lectura y búsquedas (ej. Redis, Elasticsearch).
* Permite el escalado independiente asimétrico (normalmente hay más lecturas que escrituras).
