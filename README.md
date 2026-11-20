# Data-science-AI
import sqlite3
import numpy as np
from sklearn.linear_model import LinearRegression

# 1. إنشاء قاعدة بيانات وتخزين المؤشرات التاريخية
conn = sqlite3.connect("trends_ai.db")
cursor = conn.cursor()

cursor.execute("""
CREATE TABLE IF NOT EXISTS market_metrics (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    period INTEGER,
    value REAL
)
""")

# بيانات تجريبية (فترة زمنية + قيمة المؤشر)
sample_data = [
    (1, 100.0),
    (2, 104.5),
    (3, 102.0),
    (4, 108.0),
    (5, 113.5),
    (6, 111.0),
    (7, 118.0),
]

cursor.execute("DELETE FROM market_metrics")
cursor.executemany("INSERT INTO market_metrics (period, value) VALUES (?, ?)", sample_data)
conn.commit()

# 2. قراءة البيانات من قاعدة البيانات
cursor.execute("SELECT period, value FROM market_metrics ORDER BY period ASC")
rows = cursor.fetchall()

periods = np.array([r[0] for r in rows]).reshape(-1, 1)
values = np.array([r[1] for r in rows])

# 3. تدريب نموذج ذكاء اصطناعي بسيط للتنبؤ بالاتجاه والقيمة القادمة
model = LinearRegression()
model.fit(periods, values)

next_period = np.array([[len(rows) + 1]])
predicted_next_value = float(model.predict(next_period)[0])

# تحديد هل المؤشر طالع (Upward) ولا نازل (Downward)
last_value = float(values[-1])
trend = "📈 طالع (Upward)" if predicted_next_value >= last_value else "📉 نازل (Downward)"
change_pct = ((predicted_next_value - last_value) / last_value) * 100

print(f"آخر قيمة مسجلة: {last_value:.2f}")
print(f"التنبؤ للفترة القادمة: {predicted_next_value:.2f}")
print(f"حالة المؤشر: {trend} ({change_pct:+.2f}%)")

conn.close()
