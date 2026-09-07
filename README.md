# Flask-kitob-API-test_client-bilan-endpoint-testlari
app.py — Flask ilovasi:

Python
from flask import Flask, jsonify, request

app = Flask(__name__)

# Vaqtinchalik ma'lumotlar ombori
books_db = [{"id": 1, "nom": "O'tkan kunlar", "muallif": "Abdulla Qodiriy"}]


@app.route("/books", methods=["GET"])
def get_books():
    return jsonify(books_db), 200


@app.route("/books", methods=["POST"])
def create_book():
    data = request.get_json()

    # Validatsiya: 'nom' maydoni majburiy
    if not data or "nom" not in data:
        return jsonify({"error": "'nom' maydoni ko'rsatilishi shart"}), 400

    new_book = {
        "id": len(books_db) + 1,
        "nom": data["nom"],
        "muallif": data.get("muallif", "Noma'lum"),
    }
    books_db.append(new_book)
    return jsonify(new_book), 201
conftest.py — Test mijozi fixture'i:

Python
import pytest
from app import app as flask_app


@pytest.fixture
def client():
    # Test rejimini yoqish
    flask_app.config["TESTING"] = True

    with flask_app.test_client() as client:
        yield client
test_app.py — Pytest orqali API testlari:

Python
# --- GET /books testlari ---


def test_get_books_success(client):
    """Barcha kitoblarni olish testi (Muvaffaqiyatli - 200)"""
    response = client.get("/books")

    assert response.status_code == 200
    data = response.get_json()
    assert isinstance(data, list)
    assert len(data) >= 1


def test_get_books_invalid_route(client):
    """Mavjud bo'lmagan marshrutga so'rov testi (Xato - 404)"""
    response = client.get("/books_not_found")

    assert response.status_code == 404


# --- POST /books testlari ---


def test_create_book_success(client):
    """Yangi kitob qo'shish testi (Muvaffaqiyatli - 201)"""
    payload = {"nom": "Mehrobdan chayon", "muallif": "Abdulla Qodiriy"}

    response = client.post("/books", json=payload)

    assert response.status_code == 201
    data = response.get_json()
    assert data["nom"] == "Mehrobdan chayon"
    assert "id" in data


def test_create_book_missing_nom_validation(client):
    """'nom' maydoni bo'lmagan holda kitob qo'shish testi (Validatsiya xatosi - 400)"""
    payload = {"muallif": "Otkir Hoshimov"}  # 'nom' yo'q

    response = client.post("/books", json=payload)

    assert response.status_code == 400
    data = response.get_json()
    assert data["error"] == "'nom' maydoni ko'rsatilishi shart"
