from flask import Flask, render_template, request
import pandas as pd

app = Flask(__name__)

# load dữ liệu dân cư mô phỏng
citizen = pd.read_csv("citizens.csv")

def lookup(idnum):
    row = citizen[citizen["national_id"] == str(idnum)]
    if row.empty:
        return None
    return row.iloc[0].to_dict()

@app.route("/")
def index():
    return render_template("index.html")

@app.route("/analyze", methods=["POST"])
def analyze():
    name = request.form.get("name")
    idnum = request.form.get("idnum")
    score = float(request.form.get("score"))

    cz = lookup(idnum)
    status = cz["household_status"] if cz else "Không có dữ liệu"
    distance = float(cz["distance_km"]) if cz else 0

    penalty = 0
    if status == "poor": penalty += 8
    if distance > 10: penalty += 6

    final = max(score - penalty, 0)

    return render_template(
        "result.html",
        name=name,
        idnum=idnum,
        score=score,
        status=status,
        distance=distance,
        final=final,
    )

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=10000)
