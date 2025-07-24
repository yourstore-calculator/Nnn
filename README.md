# Nnn
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8">
  <title>حاسبة الأسعار</title>
  <style>
    body {
      background-color: #1e1e1e;
      color: #ffffff;
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      padding: 20px;
    }
    label, select, input {
      display: block;
      margin: 10px 0;
    }
    select, input[type="number"] {
      padding: 10px;
      width: 300px;
      border: none;
      border-radius: 5px;
    }
    button {
      padding: 10px 20px;
      background-color: #4CAF50;
      border: none;
      border-radius: 5px;
      color: white;
      cursor: pointer;
      margin-top: 15px;
      margin-left: 10px;
    }
    .results {
      margin-top: 30px;
      border-top: 1px solid #ccc;
      padding-top: 20px;
    }
    .result-box {
      background-color: #333;
      border: 1px solid #555;
      padding: 15px;
      margin-bottom: 15px;
      border-radius: 8px;
    }
  </style>
</head>
<body>

  <h1>حاسبة أسعار المنتجات</h1>

  <label>النوع الرئيسي:</label>
  <select id="mainType" onchange="populateSubTypes()">
    <option value="">-- اختر --</option>
    <option value="نوافذ">نوافذ</option>
    <option value="أبواب">أبواب</option>
    <option value="أبواب السحب">أبواب السحب</option>
  </select>

  <label>النوع الفرعي:</label>
  <select id="subType"></select>

  <label>الطول (متر):</label>
  <input type="number" id="lengthInput" placeholder="مثال: 2.2" step="0.01">

  <label>العرض (متر):</label>
  <input type="number" id="widthInput" placeholder="مثال: 1" step="0.01">

  <label>الكمية:</label>
  <input type="number" id="quantityInput" value="1" min="1">

  <button onclick="calculate()">احسب</button>
  <button onclick="clearResults()">محو النتائج</button>

  <div class="results" id="results"></div>

  <script>
    const prices = {
      "دبل جلاس دبل فريم": { price: 34, per: "m", factor: 0.13 },
      "دبل جلاس سنجل فريم": { price: 26, per: "m", factor: 0.13 },
      "سنجل جلاس سنجل فريم": { price: 20, per: "m", factor: 0.13 },
      "نوافذ السلايدنج": { price: 10, per: "m", factor: 0.13, isSliding: true },
      "النوافذ الكهربائية": { price: 102, per: "m", factor: 0.13 },
      "سكاي لايت - بدون مكينة": { price: 56, per: "m", factor: 0.13 },
      "سكاي لايت - مع مكينة": { price: 145, per: "m", factor: 0.13 },
      "كارتن وول - ثقيل": { price: 56, per: "m", factor: 0.13 },
      "كارتن وول - خفيف": { price: 45, per: "m", factor: 0.13 },
      "باب المدخل - زينك": { price: 66, per: "m", factor: 0.2, add: 10 },
      "باب المدخل - ستينلس ستيل": { price: 120, per: "m", factor: 0.2, add: 10 },
      "باب المدخل - كاست المنيوم": { price: 168, per: "m", factor: 0.2, add: 10 },
      "باب WPC": { price: 45, per: "unit", factor: 0.11 },
      "باب ألمنيوم": { price: 65, per: "unit", factor: 0.11 },
      "باب دورة مياه": { price: 55, per: "unit", factor: 0.11 },
      "سحب داخلي": { price: 38, per: "m", factor: 0.13 },
      "سحب خارجي": { price: 55, per: "m", factor: 0.13 },
      "سحب WPC": { price: 61, per: "m", factor: 0.11 },
      "شبك ثابت": { price: 20, per: "m", factor: 0.05 },
      "شبك سلايد": { price: 22, per: "m", factor: 0.05 },
      "شبك فولدينج": { price: 25, per: "m", factor: 0.05 },
      "شبك باب": { price: 28, per: "m", factor: 0.05 },
      "ستارة داخلية": { price: 26, per: "m", factor: 0.0 }
    };

    const subTypeMap = {
      "نوافذ": [
        "دبل جلاس دبل فريم", "دبل جلاس سنجل فريم", "سنجل جلاس سنجل فريم",
        "نوافذ السلايدنج", "النوافذ الكهربائية",
        "سكاي لايت - بدون مكينة", "سكاي لايت - مع مكينة",
        "كارتن وول - ثقيل", "كارتن وول - خفيف",
        "ستارة داخلية", "شبك ثابت", "شبك سلايد", "شبك فولدينج", "شبك باب"
      ],
      "أبواب": [
        "باب المدخل - زينك", "باب المدخل - ستينلس ستيل", "باب المدخل - كاست المنيوم",
        "باب WPC", "باب ألمنيوم", "باب دورة مياه"
      ],
      "أبواب السحب": [
        "سحب داخلي", "سحب خارجي", "سحب WPC"
      ]
    };

    function populateSubTypes() {
      const mainType = document.getElementById("mainType").value;
      const subTypeSelect = document.getElementById("subType");
      subTypeSelect.innerHTML = "";
      if (subTypeMap[mainType]) {
        subTypeMap[mainType].forEach(type => {
          const option = document.createElement("option");
          option.value = type;
          option.text = type;
          subTypeSelect.appendChild(option);
        });
      }
    }

    function calculate() {
      const type = document.getElementById("subType").value;
      const length = parseFloat(document.getElementById("lengthInput").value);
      const width = parseFloat(document.getElementById("widthInput").value);
      const quantity = parseInt(document.getElementById("quantityInput").value);
      const data = prices[type];

      if (!data || isNaN(length) || isNaN(width) || isNaN(quantity)) {
        alert("يرجى تعبئة كل الحقول بشكل صحيح.");
        return;
      }

      const area = length * width;
      let base = data.per === "m" ? area * data.price : data.price;
      if (data.isSliding) base += 10;
      if (data.add) base += data.add;

      const shipping = area * data.factor * 48;
      const total = (base + shipping) * quantity;

      const resultsDiv = document.getElementById("results");
      const resultBox = document.createElement("div");
      resultBox.className = "result-box";
      resultBox.innerHTML = `
        <strong>النوع:</strong> ${type}<br>
        <strong>المقاس:</strong> ${length} × ${width} متر<br>
        <strong>الكمية:</strong> ${quantity}<br>
        <strong>سعر الشحن:</strong> ${shipping.toFixed(2)} ريال<br>
        <strong>السعر النهائي:</strong> ${total.toFixed(2)} ريال
      `;
      resultsDiv.appendChild(resultBox);
    }

    function clearResults() {
      document.getElementById("results").innerHTML = "";
    }
  </script>

</body>
</html>
