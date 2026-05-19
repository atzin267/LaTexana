import makeWASocket, { DisconnectReason, useMultiFileAuthState } from '@whiskeysockets/baileys'
import qrcode from 'qrcode-terminal'
import fs from 'fs'
import path from 'path'

const PRODUCTOS = {
  sabritas: [
    { nombre: 'Sabritas Sal', precio: 20 },
    { nombre: 'Sabritas Limón', precio: 20 },
    { nombre: 'Rancheritos', precio: 17 }
  ],
  chettos: [
    { nombre: 'Chettos Bolita', precio: 17 },
    { nombre: 'Chettos Vampiro', precio: 17 }
  ],
  rancheritos: [
    { nombre: 'Rancheritos', precio: 17 }
  ],
  pepsi: [
    { nombre: 'Pepsi 500ml', precio: 15 },
    { nombre: 'Pepsi 2L', precio: 35 }
  ],
  coca: [
    { nombre: 'Coca Cola 500ml', precio: 15 },
    { nombre: 'Coca Cola 2L', precio: 35 }
  ]
}

function buscarProductos(palabra) {
  const palabraNormalizada = palabra.toLowerCase().trim()
  let resultados = []

  for (const [categoria, productos] of Object.entries(PRODUCTOS)) {
    if (categoria.includes(palabraNormalizada) || palabraNormalizada.includes(categoria)) {
      resultados.push(...productos)
    } else {
      const encontrados = productos.filter(p => 
        p.nombre.toLowerCase().includes(palabraNormalizada)
      )
      resultados.push(...encontrados)
    }
  }

  return [...new Set(resultados.map(JSON.stringify))].map(JSON.parse)
}

function formatearRespuesta(productos) {
  if (productos.length === 0) {
    return '❌ No encontré productos con esa búsqueda.\n\n📝 Intenta con:\n- Sabritas\n- Chettos\n- Rancheritos\n- Pepsi\n- Coca'
  }

  let mensaje = '🛍️ *Productos encontrados:*\n\n'
  productos.forEach((p, i) => {
    mensaje += `${i + 1}. ${p.nombre} - $${p.precio}\n`
  })
  mensaje += '\n📲 Escribe otro producto para buscar'
  return mensaje
}

async function iniciarBot() {
  const { state, saveCreds } = await useMultiFileAuthState('auth_info_baileys')

  const sock = makeWASocket({
    auth: state,
    printQRInTerminal: true
  })

  sock.ev.on('connection.update', (update) => {
    const { connection, lastDisconnect, qr } = update

    if (qr) {
      console.log('📱 Escanea este código QR con WhatsApp:\n')
      qrcode.generate(qr, { small: true })
    }

    if (connection === 'open') {
      console.log('✅ Bot conectado exitosamente!')
      console.log('📱 Ahora puedes enviar mensajes al bot')
    } else if (connection === 'close') {
      const shouldReconnect =
        (lastDisconnect?.error)?.output?.statusCode !== DisconnectReason.loggedOut
      console.log('❌ Desconectado:', lastDisconnect?.error)
      if (shouldReconnect) {
        setTimeout(() => iniciarBot(), 3000)
      }
    }
  })

  sock.ev.on('creds.update', saveCreds)

  sock.ev.on('messages.upsert', async (m) => {
    const message = m.messages[0]

    if (!message.message) return

    const from = message.key.remoteJid
    const texto = message.message.conversation || 
                  message.message.extendedTextMessage?.text || ''

    if (message.key.fromMe) return

    console.log(`\n💬 Mensaje de ${from}: ${texto}`)

    const productos = buscarProductos(texto)
    const respuesta = formatearRespuesta(productos)

    await sock.sendMessage(from, { text: respuesta })
    console.log(`✅ Respuesta enviada a ${from}`)
  })
}

iniciarBot().catch(console.error)