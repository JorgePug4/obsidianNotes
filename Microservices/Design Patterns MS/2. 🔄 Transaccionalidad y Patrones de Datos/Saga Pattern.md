Gestiona transacciones distribuidas mediante una secuencia de transacciones locales independientes.
* Si un proceso en la cadena falla, el patrón ejecuta **Transacciones Compensatorias** para realizar un "rollback" lógico en los servicios que ya habían completado su parte.
* **Orquestación:** Un controlador centralizado dirige la ejecución.
* **Coreografía:** Basado en eventos, donde cada servicio reacciona de forma autónoma.