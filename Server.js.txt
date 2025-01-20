const express = require('express');
const http = require('http');
const socketIo = require('socket.io');

const app = express();
const server = http.createServer(app);
const io = socketIo(server);

// Servir archivos estáticos (HTML, JS, etc.)
app.use(express.static('public'));

// Manejo de eventos de conexión de Socket.IO
io.on('connection', (socket) => {
  console.log('Nuevo cliente conectado');
  
  // Enviar y recibir mensajes para la señalización
  socket.on('signal', (data) => {
    socket.broadcast.emit('signal', data); // Reenvía la señal a todos los clientes excepto el remitente
  });

  socket.on('disconnect', () => {
    console.log('Cliente desconectado');
  });
});

// Puerto en el que se ejecuta el servidor
const PORT = process.env.PORT || 3000;
server.listen(PORT, () => {
  console.log(`Servidor de señalización corriendo en http://localhost:${PORT}`);
});
