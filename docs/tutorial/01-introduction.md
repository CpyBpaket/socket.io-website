// Подключение необходимых библиотек
const express = require("express");
const http = require("http");
const socketIo = require("socket.io");

// Инициализация сервера
const app = express();
const server = http.createServer(app);
const io = socketIo(server);

// Хранилище для сообщений в памяти
let messages = [];

// Обработка подключений
io.on("connection", (socket) => {
    console.log("Пользователь подключился");

    // Отправка всех старых сообщений новому пользователю
    socket.emit("previousMessages", messages);

    // Обработка получения нового сообщения
    socket.on("sendMessage", (message) => {
        messages.push(message);
        io.emit("newMessage", message); // Отправка сообщения всем пользователям
    });

    socket.on("disconnect", () => {
        console.log("Пользователь отключился");
    });
});

// Обработка запроса на главную страницу
app.get("/", (req, res) => {
    res.send(`
        <!DOCTYPE html>
        <html lang="en">
        <head>
            <meta charset="UTF-8">
            <title>Мессенджер</title>
            <script src="/socket.io/socket.io.js"></script>
        </head>
        <body>
            <div id="messages"></div>
            <input id="messageInput" type="text" placeholder="Введите сообщение">
            <button onclick="sendMessage()">Отправить</button>

            <script>
                const socket = io();

                // Отображение старых сообщений
                socket.on("previousMessages", (messages) => {
                    messages.forEach((message) => {
                        displayMessage(message);
                    });
                });

                // Отображение новых сообщений
                socket.on("newMessage", (message) => {
                    displayMessage(message);
                });

                // Отправка нового сообщения
                function sendMessage() {
                    const input = document.getElementById("messageInput");
                    const message = input.value;
                    socket.emit("sendMessage", message);
                    input.value = "";
                }

                function displayMessage(message) {
                    const messagesDiv = document.getElementById("messages");
                    const messageElement = document.createElement("div");
                    messageElement.textContent = message;
                    messagesDiv.appendChild(messageElement);
                }
            </script>
        </body>
        </html>
    `);
});

// Запуск сервера
const PORT = process.env.PORT || 3000;
server.listen(PORT, () => {
    console.log(`Сервер запущен на http://localhost:${PORT}`);
});
