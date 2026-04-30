Previene fallos en cascada gestionando estados para proteger servicios degradados.
* 🟢 **Cerrado:** Peticiones fluyen con normalidad.
* 🔴 **Abierto:** Umbral de fallos superado. Las peticiones fallan rápido (*Fast-Fail*) sin llamar al servicio destino, dándole tiempo de recuperación.
* 🟡 **Medio-Abierto:** Permite un tráfico mínimo de prueba para verificar si el servicio destino ya es estable.
> 💡 *Implementación:* Frecuentemente implementado con **Polly** en el ecosistema .NET.