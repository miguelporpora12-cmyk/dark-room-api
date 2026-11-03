
import sqlite3
import secrets
from uuid import uuid4
from datetime import datetime, timezone

from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, Field

app = FastAPI(
    title="Dark Room API",
    description="API de gerenciamento de pedidos de salas",
    version="1.1.0"
)

DATABASE = "rooms.db"

MODOS = [
    "Full Ouro",
    "Gel Infinito",
    "Full Capa",
    "Com Carregamento",
    "Sem Carregamento",
    "Tático",
    "UMP e XM8"
]


def conectar():
    return sqlite3.connect(DATABASE)


def iniciar_banco():
    with conectar() as db:
        db.execute("""
            CREATE TABLE IF NOT EXISTS rooms (
                request_id TEXT PRIMARY KEY,
                mode TEXT NOT NULL,
                suggested_password TEXT NOT NULL,
                status TEXT NOT NULL,
                created_at TEXT NOT NULL
            )
        """)


iniciar_banco()


class RoomRequest(BaseModel):
    mode: str = Field(min_length=1, max_length=40)


@app.get("/")
def inicio():
    return {
        "name": "Dark Room API",
        "version": "1.1.0",
        "status": "online"
    }


@app.get("/health")
def health():
    return {"status": "ok"}


@app.post("/rooms")
def criar_pedido(dados: RoomRequest):

    if dados.mode not in MODOS:
        raise HTTPException(
            status_code=400,
            detail={
                "erro": "Modo inválido",
                "modos_disponiveis": MODOS
            }
        )

    request_id = str(uuid4())
    senha_sugerida = f"{secrets.randbelow(100):02d}"
    data = datetime.now(timezone.utc).isoformat()

    with conectar() as db:
        db.execute("""
            INSERT INTO rooms (
                request_id,
                mode,
                suggested_password,
                status,
                created_at
            )
            VALUES (?, ?, ?, ?, ?)
        """, (
            request_id,
            dados.mode,
            senha_sugerida,
            "awaiting_provider",
            data
        ))

    return {
        "success": True,
        "message": "Pedido registrado.",
        "request_id": request_id,
        "mode": dados.mode,
        "suggested_password": senha_sugerida,
        "status": "awaiting_provider",
        "room_created": False,
        "details": "A integração autorizada com o Free Fire ainda não foi configurada."
    }


@app.get("/rooms/{request_id}")
def consultar_pedido(request_id: str):

    with conectar() as db:
        db.row_factory = sqlite3.Row
        resultado = db.execute(
            "SELECT * FROM rooms WHERE request_id = ?",
            (request_id,)
        ).fetchone()

    if resultado is None:
        raise HTTPException(
            status_code=404,
            detail="Pedido não encontrado."
        )

    return {
        "request_id": resultado["request_id"],
        "mode": resultado["mode"],
        "suggested_password": resultado["suggested_password"],
        "status": resultado["status"],
        "created_at": resultado["created_at"],
        "room_created": False
    }
