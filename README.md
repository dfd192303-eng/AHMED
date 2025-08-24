wish-system/
├─ server.js
├─ package.json
└─ public/
    ├─ index.html       ← واجهة الجمهور
    ├─ display.html     ← شاشة العرض
    ├─ style.css        ← الأنماط المشتركة
    └─ logo.png         ← شعار الحفل
const express = require('express');
const http = require('http');
const socketIo = require('socket.io');
const twilio = require('twilio');

const app = express();
const server = http.createServer(app);
const io = socketIo(server);

app.use(express.static('public'));

const accountSid = process.env.TWILIO_SID;
const authToken = process.env.TWILIO_TOKEN;
const twilioNumber = process.env.TWILIO_NUMBER;
const myNumber = process.env.MY_NUMBER;

const client = twilio(accountSid, authToken);

let usedNumbers = {}; // رقم: أمنية

io.on('connection', socket => {
  socket.emit('updateNumbers', usedNumbers);

  socket.on('newWish', async ({number, wish}) => {
    if(usedNumbers[number]) return;
    usedNumbers[number] = wish;
    io.emit('updateNumbers', usedNumbers);

    // إرسال SMS إذا متوفر Twilio
    if(accountSid && authToken && twilioNumber && myNumber){
      try{
        await client.messages.create({
          body: `تم إضافة أمنية جديدة: رقم ${number} - ${wish}`,
          from: twilioNumber,
          to: myNumber
        });
      } catch(err){
        console.error('خطأ في إرسال SMS:', err.message);
      }
    }
  });
});

const PORT = process.env.PORT || 3000;
server.listen(PORT, () => console.log(`Server running on port ${PORT}`));
