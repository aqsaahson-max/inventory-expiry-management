# Invigo FreshGuard

Invigo FreshGuard is an AI-assisted inventory management system that helps retailers identify perishable stock likely to expire before it sells. It combines batch inventory and sales history to estimate expiry risk, flag batches that need attention, and suggest discounts for staff to review.

## What It Does

- **Batch inventory:** Tracks quantities and expiry dates for each batch of a product.
- **Sales recording:** Updates stock using First Expire, First Out (FEFO), allocating sales to the earliest-expiring eligible batches.
- **Expiry-risk prediction:** Uses inventory and sales features, such as shelf life and average daily sales, to estimate the risk of stock expiring unsold.
- **Alerts and discounts:** Flags at-risk batches and generates discount suggestions. Discounts require approval before they can be applied at checkout.
- **Waste tracking:** Distinguishes expired stock awaiting action from waste that has already been recorded.

## My Contribution: Sales Recording

This was a two-person project. I built the Sales Recording module, which connects checkout transactions to batch-level inventory. The cashier selects a product and quantity, and the module uses FEFO to determine which batches to deduct from. If a sale spans multiple batches, the module splits the allocation and applies approved discounts only to units from the discounted batch.

The sales history captured by this module also supports reporting and provides the demand data used to calculate features for expiry-risk prediction.

## Architecture

| Component | Technology | Location |
| --- | --- | --- |
| Frontend | React + Vite | `src/` |
| Backend | Java + Spring Boot | `Invigo/` |
| ML prediction service and notebooks | Python + FastAPI | `AIML/` |

The Spring Boot backend prepares features from inventory and sales records and sends them to the FastAPI prediction service. The service returns a risk label and probability for each batch. The backend stores the prediction and applies business rules to generate alerts or discount suggestions. The `AIML/` notebooks document the workflow from data preparation and feature engineering through model training and evaluation.

## Run Locally

Start the database first, then the ML service, the Spring Boot backend, and the frontend. Run each service in a separate terminal.

### 1. ML Service — Port 8000

From `AIML/`, create a virtual environment and install the dependencies. These commands are for Windows PowerShell:

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
```

Make sure the trained model artifact is available at the path expected by the service. If your copy does not include the artifact, use the notebooks in `AIML/` to reproduce it, then set `MODEL_PATH` to the resulting file:

```powershell
$env:MODEL_PATH = "C:\path\to\expiry_risk_model.pkl"
.\.venv\Scripts\python.exe -m uvicorn main:app --reload --port 8000
```

Endpoints: `GET /`, `GET /health`, `POST /predict`, and `POST /predict-batch`.

### 2. Backend — Port 8080

Use `Invigo/.env.example` as a reference and set the required environment variables in the terminal where you run the backend. Key settings include `DB_URL`, `DB_USERNAME`, `DB_PASSWORD`, `JWT_SECRET`, `APP_FRONTEND_BASE_URL`, `APP_CORS_ALLOWED_ORIGINS`, and `ML_API_URL`. Use the mail settings in the example if you are configuring email onboarding.

From `Invigo/`, run:

```powershell
.\mvnw.cmd spring-boot:run
```

On macOS or Linux, use `./mvnw spring-boot:run`.

### 3. Frontend — Port 5173

From the repository root, create a local `.env` file using `.env.example` as a reference:

```env
VITE_API_BASE_URL=http://localhost:8080/api
```

Then run:

```bash
npm install
npm run dev
```

Open `http://localhost:5173`.

## Verification

Run the frontend checks from the repository root:

```bash
npm test
npm run lint
```

Run the backend tests from `Invigo/`:

```powershell
.\mvnw.cmd test
```

On macOS or Linux, use `./mvnw test`.

## Security and Maintenance

Backend credentials are supplied through environment variables. The backend determines user roles at sign-in, and new staff receive password setup instructions by email. The frontend uses shared API configuration, and the ML service provides a health endpoint for checking that it is running.

## Credits

Invigo FreshGuard was built as a two-person team project. This portfolio fork preserves the original commit history. See the [original repository](https://github.com/IT24104036/Inventory-Management-System-) for the shared project.
