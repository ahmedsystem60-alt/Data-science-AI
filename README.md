# Data-science-AI
import sqlite3
import numpy as np

# 1. تجهيز البيانات بمؤشرات متنوعة
conn = sqlite3.connect("market_dashboard.db")
cursor = conn.cursor()

cursor.execute("""
CREATE TABLE IF NOT EXISTS metrics (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    indicator_name TEXT,
    current_value REAL,
    previous_value REAL
)
""")

sample_metrics = [
    ("لآلئ البيانات (Data Yield)", 1250.0, 1100.0),
    ("معدل المعالجة (Throughput)", 98.5, 95.0),
    ("زمن الاستجابة (Latency ms)", 12.4, 15.1),
    ("دقة التنبؤ (Prediction Accuracy)", 0.88, 0.82),
    ("حجم التدفق (Stream Volume)", 4500, 4800)
]

cursor.execute("DELETE FROM metrics")
cursor.executemany("INSERT INTO metrics (indicator_name, current_value, previous_value) VALUES (?, ?, ?)", sample_metrics)
conn.commit()

# 2. جراء الحسابات وتنسيق الجدول
cursor.execute("SELECT indicator_name, current_value, previous_value FROM metrics")
rows = cursor.fetchall()

print("="*65)
print(f"{'المؤشر':<30} | {'الحالي':<10} | {'التغير %':<10} | {'الحالة'}")
print("-"*65)

for name, curr, prev in rows:
    change = ((curr - prev) / abs(prev)) * 100
    # تحديد الاتجاه بناءً على طبيعة المؤشر (بعضها الأفضل فيه أقل)
    if "Latency" in name:
        trend = "🟢 ممتاز" if change <= 0 else "🔴 مرتفع"
    else:
        trend = "🟢 صاعد" if change >= 0 else "🔴 هابط"
        
    print(f"{name:<30} | {curr:<10.1f} | {change:<+10.2f}% | {trend}")

print("="*65)

conn.close()
