# 🏗️ AI Supplier Ranking System (Laravel + Python)

This guide shows how to implement an **AI-based supplier ranking system** in your Laravel project using **XGBoost** and a **Python microservice**.

✅ Works with **Laravel hosted on AWS EC2**  
✅ Uses **MySQL** (already in your EC2)  
✅ No GPU required — runs on CPU  
✅ Easy to maintain & retrain automatically

---

## 1️⃣ System Architecture

```mermaid
flowchart LR
A[Laravel App + MySQL] -->|Fetch Supplier Data| B[Python Training Script]
B -->|Generate supplier_model.json| C[Model Storage (/var/www/models)]
A -->|API Call| D[FastAPI Prediction Service]
D -->|Returns Ranking| A
```

- **Laravel**: Manages inventory, suppliers, and user interface.
- **Python**: Trains the model with supplier data using XGBoost.
- **FastAPI**: Exposes a simple HTTP API to make predictions.

---

## 2️⃣ Setup Instructions

### Step 1: Install Python on EC2

```bash
sudo apt update && sudo apt install -y python3 python3-pip
python3 -m pip install --upgrade pip
```

Install required libraries:

```bash
pip install xgboost pandas fastapi uvicorn scikit-learn mysql-connector-python
```

---

### Step 2: Create Model Training Script

Create `train_model.py` in `/var/www/supplier_ai/`:

```python
import pandas as pd
import mysql.connector
from xgboost import XGBClassifier
import joblib

# 1. Connect to MySQL
db = mysql.connector.connect(
    host="127.0.0.1",
    user="your_mysql_user",
    password="your_mysql_password",
    database="your_db_name"
)

query = """
SELECT supplier_id, total_orders, on_time_delivery_rate, defect_rate, average_cost, payment_terms_score, quality_score
FROM suppliers;
"""
df = pd.read_sql(query, db)

# 2. Define features and label (example: 1=good supplier, 0=bad)
X = df[["total_orders", "on_time_delivery_rate", "defect_rate", "average_cost", "payment_terms_score", "quality_score"]]
y = (df["quality_score"] > 70).astype(int)

# 3. Train Model
model = XGBClassifier()
model.fit(X, y)

# 4. Save Model
joblib.dump(model, "/var/www/supplier_ai/supplier_model.json")
print("✅ Model trained and saved!")
```

---

### Step 3: Create FastAPI Prediction Service

Create `app.py` in `/var/www/supplier_ai/`:

```python
from fastapi import FastAPI
import joblib
import pandas as pd

app = FastAPI()
model = joblib.load("/var/www/supplier_ai/supplier_model.json")

@app.post("/predict")
async def predict(supplier: dict):
    X = pd.DataFrame([supplier])
    prediction = model.predict_proba(X)[0][1]  # probability of being valuable
    return {"score": float(prediction)}
```

Run the service:

```bash
uvicorn app:app --host 0.0.0.0 --port 8001
```

Keep it running using **systemd** or **pm2**.

---

### Step 4: Laravel Integration

In Laravel, create a controller method to call the FastAPI service:

```php
use Illuminate\Support\Facades\Http;

public function getSupplierScore($supplierId)
{
    $supplier = \DB::table('suppliers')->find($supplierId);

    $response = Http::post('http://127.0.0.1:8001/predict', [
        'total_orders' => $supplier->total_orders,
        'on_time_delivery_rate' => $supplier->on_time_delivery_rate,
        'defect_rate' => $supplier->defect_rate,
        'average_cost' => $supplier->average_cost,
        'payment_terms_score' => $supplier->payment_terms_score,
        'quality_score' => $supplier->quality_score,
    ]);

    return $response->json();
}
```

---

### Step 5: Automate Model Retraining

Add a Laravel command `php artisan make:command RetrainSupplierModel`:

```php
public function handle()
{
    exec("python3 /var/www/supplier_ai/train_model.py", $output, $resultCode);
    if ($resultCode === 0) {
        $this->info('✅ Supplier model retrained successfully.');
    } else {
        $this->error('❌ Model retraining failed.');
    }
}
```

Schedule it in `app/Console/Kernel.php`:

```php
protected function schedule(Schedule $schedule)
{
    $schedule->command('supplier:retrain')->monthly();
}
```

This will retrain the model automatically every month.

---

### Step 6: Secure Your Setup

- Restrict FastAPI endpoint to localhost (`127.0.0.1`) or internal VPC.
- Use Laravel policies to ensure only admins can trigger retraining.
- Backup your `supplier_model.json` after every retrain.

---

## 3️⃣ Example API Usage

```bash
curl -X POST http://127.0.0.1:8001/predict -H "Content-Type: application/json" -d '{
  "total_orders": 500,
  "on_time_delivery_rate": 0.95,
  "defect_rate": 0.02,
  "average_cost": 150,
  "payment_terms_score": 8,
  "quality_score": 85
}'
```

Response:

```json
{
  "score": 0.92
}
```

Meaning: 92% chance that this supplier is valuable.

---

## 4️⃣ Benefits of This Setup

- **Lightweight** – CPU-based training
- **Cost-effective** – Runs on your existing EC2
- **Automated** – Monthly retraining
- **Expandable** – Add more features as your data grows

---

## 5️⃣ Future Improvements

- Store historical scores in DB to analyze trends.
- Visualize supplier ranking in Laravel admin panel.
- Add alerts when supplier performance drops.
- Experiment with LightGBM for faster training.

---

## 6️⃣ Folder Structure

```
/var/www/
 ├── laravel_project/
 └── supplier_ai/
      ├── train_model.py
      ├── app.py
      └── supplier_model.json
```

---

## 7️⃣ Requirements

- **Laravel** 10+
- **Python** 3.9+
- **AWS EC2** (t3.medium or higher recommended)
- **MySQL** running in EC2

---

## 8️⃣ Troubleshooting

| Issue | Fix |
|------|------|
| `ModuleNotFoundError` for pandas/xgboost | Run `pip install pandas xgboost` |
| FastAPI not accessible | Check firewall or use `--host 0.0.0.0` |
| Model not updating after retraining | Restart FastAPI service to reload model |

---

✅ **You now have a working AI-powered supplier ranking system inside your Laravel project!**
