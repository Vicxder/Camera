import cv2
import asyncio
import socketio
from aiortc import RTCPeerConnection, RTCSessionDescription
from aiortc.contrib.media import MediaPlayer, MediaStreamTrack

# Establecer conexión de señalización
sio = socketio.Client()

@sio.event
def connect():
    print('Conectado al servidor de señalización')

@sio.event
def signal(data):
    handle_signal(data)

def handle_signal(data):
    if 'sdp' in data:
        pc.setRemoteDescription(RTCSessionDescription(sdp=data['sdp']['sdp'], type=data['sdp']['type']))
        if data['sdp']['type'] == 'offer':
            create_answer()
    elif 'ice' in data:
        pc.addIceCandidate(data['ice'])

def send_signal(sdp=None, ice=None):
    if sdp:
        sio.emit('signal', {'sdp': {'sdp': sdp.sdp, 'type': sdp.type}})
    if ice:
        sio.emit('signal', {'ice': ice})

# Crear la conexión WebRTC
pc = RTCPeerConnection()

# Crear pista de video
class VideoTrack(MediaStreamTrack):
    kind = "video"

    def __init__(self, frame_rate=30):
        super().__init__()
        self.frame_rate = frame_rate
        self.cap = cv2.VideoCapture(0)

    async def recv(self):
        ret, frame = self.cap.read()
        if not ret:
            raise Exception("No se pudo leer la cámara")
        # Convertir la imagen a un formato adecuado para WebRTC
        return frame

# Configurar y enviar la oferta de WebRTC
async def create_offer():
    video_track = VideoTrack()
    pc.addTrack(video_track)

    offer = await pc.createOffer()
    await pc.setLocalDescription(offer)

    send_signal(sdp=offer)

# Configurar el cliente WebRTC
sio.connect('http://localhost:3000')

# Iniciar la transmisión
asyncio.get_event_loop().run_until_complete(create_offer())
