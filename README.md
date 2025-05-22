# 24solver
<!DOCTYPE html><html lang="zh-Hant">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>24 點萬用解決器（手機版）</title>
  <style>
    body {
      font-family: sans-serif;
      background-color: #f2f2f2;
      margin: 0;
      padding: 20px;
      text-align: center;
    }
    h2 {
      margin-bottom: 10px;
    }
    .input-group {
      display: flex;
      justify-content: center;
      flex-wrap: wrap;
      gap: 10px;
      margin: 10px 0;
    }
    input {
      width: 60px;
      padding: 10px;
      font-size: 18px;
      border-radius: 8px;
      border: 1px solid #ccc;
    }
    button {
      padding: 12px 24px;
      font-size: 18px;
      margin-top: 10px;
      border: none;
      background-color: #007bff;
      color: white;
      border-radius: 8px;
      cursor: pointer;
    }
    button:hover {
      background-color: #0056b3;
    }
    #result {
      margin-top: 20px;
      white-space: pre-line;
      font-size: 16px;
      text-align: left;
      background: white;
      padding: 10px;
      border-radius: 8px;
      max-height: 300px;
      overflow-y: auto;
    }
  </style>
</head>
<body>
  <h2>24 點解決器（支援平方、根號、階乘）</h2>
  <p>輸入 4 個數字（J=11, Q=12, K=13）</p>
  <div class="input-group">
    <input type="number" id="n1" placeholder="1">
    <input type="number" id="n2" placeholder="2">
    <input type="number" id="n3" placeholder="3">
    <input type="number" id="n4" placeholder="4">
  </div>
  <button onclick="solve()">解出 24 點</button>
  <div id="result"></div>  <script>
    function factorial(n) {
      if (n < 0 || n > 10) return null;
      let res = 1;
      for (let i = 2; i <= n; i++) res *= i;
      return res;
    }

    function variations(n) {
      const vars = new Set();
      vars.add(n);
      if (n >= 0) vars.add(Math.sqrt(n));
      if (Number.isInteger(n) && n >= 0 && n <= 10) vars.add(factorial(n));
      vars.add(n * n);
      return Array.from(vars).filter(x => isFinite(x));
    }

    function generateExpressions(a, b, c, d, ops) {
      const [op1, op2, op3] = ops;
      return [
        `(${a}${op1}${b})${op2}${c}${op3}${d}`,
        `(${a}${op1}(${b}${op2}${c}))${op3}${d}`,
        `${a}${op1}((${b}${op2}${c})${op3}${d})`,
        `${a}${op1}(${b}${op2}(${c}${op3}${d}))`,
        `(${a}${op1}${b})${op2}(${c}${op3}${d})`
      ];
    }

    function solve() {
      const nums = [
        parseFloat(document.getElementById('n1').value),
        parseFloat(document.getElementById('n2').value),
        parseFloat(document.getElementById('n3').value),
        parseFloat(document.getElementById('n4').value)
      ];
      const ops = ['+', '-', '*', '/'];
      const resultDiv = document.getElementById('result');
      resultDiv.textContent = '';

      if (nums.some(isNaN)) {
        resultDiv.textContent = '請輸入四個有效數字';
        return;
      }

      const permute = (arr) => {
        if (arr.length <= 1) return [arr];
        const result = [];
        arr.forEach((val, idx) => {
          const rest = [...arr.slice(0, idx), ...arr.slice(idx + 1)];
          const perms = permute(rest);
          perms.forEach(p => result.push([val, ...p]));
        });
        return result;
      };

      const allOpCombos = [];
      for (let o1 of ops)
        for (let o2 of ops)
          for (let o3 of ops)
            allOpCombos.push([o1, o2, o3]);

      const seen = new Set();
      let found = false;

      for (const perm of permute(nums)) {
        const var1 = variations(perm[0]);
        const var2 = variations(perm[1]);
        const var3 = variations(perm[2]);
        const var4 = variations(perm[3]);

        for (let a of var1)
          for (let b of var2)
            for (let c of var3)
              for (let d of var4)
                for (let opset of allOpCombos)
                  for (let expr of generateExpressions(a, b, c, d, opset)) {
                    try {
                      const val = eval(expr);
                      if (Math.abs(val - 24) < 1e-6) {
                        if (!seen.has(expr)) {
                          seen.add(expr);
                          resultDiv.textContent += expr + ' = 24\n';
                          found = true;
                        }
                      }
                    } catch {}
                  }
      }

      if (!found) resultDiv.textContent = '無解';
    }
  </script></body>
</html>
