<!DOCTYPE html>
<html lang="en">
<head>
  <!-- Đặt thông tin charset cho trang HTML -->
  <meta keyset="UTF-8">
  <!-- Đặt chế độ hiển thị trên các thiết bị di động -->
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <!-- Tiêu đề của trang web -->
  <title>Giải phương trình vi phân tuyến tính cấp 1</title>
  <!-- Thêm MathJax để hiển thị các công thức toán học -->
  <script src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"></script>
  <!-- Thêm một thư viện JavaScript khác -->
  <script src="all.min.js"></script>
  <!-- Thêm reset CSS để làm lại kiểu dáng cơ bản của các thẻ HTML -->
  <link href="https://cdn.jsdelivr.net/npm/reset-css@5.0.2/reset.min.css" rel="stylesheet">
</head>
<body>
  <!-- Div chính chứa ứng dụng giải phương trình -->
  <div class="app">
    <div class="solution">
      <!-- Tiêu đề của phần giải phương trình -->
      <div class="solution__header">PHƯƠNG TRÌNH VI PHÂN TUYẾN TÍNH CẤP 1</div>
      
      <!-- Phần chứa các input cho người dùng điền các giá trị cần thiết -->
      <div class="solution__ques">
        <!-- Input cho giá trị y' -->
        <div class="solution__y-diff-box">
          <input class="solution__y-diff-input" placeholder="1">
        </div>
        <!-- Mô tả biểu thức toán học y' + P(x) y = Q(x) -->
        <p>\( \times \)</p>
        <p>\( y'\:+ \)</p>
        
        <!-- Input cho P(x) -->
        <div class="solution__p-box">
          <input class="solution__p-input" placeholder="P(x)">
        </div>
        
        <p>\( \times y\: = \)</p>
        
        <!-- Input cho Q(x) -->
        <div class="solution__q-box">
          <input class="solution__q-input" placeholder="Q(x)">
        </div>
        
        <!-- Nút bấm để gọi hàm giải phương trình -->
        <button class="run" onclick="solve()">Giải</button>
      </div>

      <!-- Phần hiển thị các bước giải -->
      <div class="solution__ans">
        <div class="step-0"></div>
        <div class="step-1"></div>
        <div class="step-2"></div>
        <div class="step-3"></div> 
        <div class="step-4"></div>
        <div class="step-5"></div>
      </div>
    </div>                    
  </div>
</body>

<style>
  /* Cấu hình kiểu dáng cho phần tiêu đề */
  .solution__header {
    padding-top: 20px;
    display: flex;
    justify-content: center;
  }
  
  /* Cấu hình kiểu dáng cho phần câu hỏi và input */
  .solution__ques {
    padding-top: 20px;
    display: flex;
    justify-content: center;
  }
  
  .solution__ques > div {
    display: flex;
    width: 50px;
    margin: 0 5px 0 5px;
  }

  /* Cấu hình cho các input */
  .solution__ques > div > input {
    width: 100%;
    border: 1px solid #6e6e6e;
    border-radius: 5px;
    cursor: pointer;
  }

  /* Khi input được chọn, thay đổi kiểu dáng viền */
  .solution__ques > div > input.selected {
    border: 2px solid #000;
  }

  /* Cấu hình kiểu dáng cho các bước giải */
  .solution__ans {
    display: grid;
    width: 100%;
    justify-content: center;
    gap: 15px;
    margin-top: 30px;
    font-size: 20px;
  }
</style>

<script>
// Khai báo các biến sẽ dùng để lưu trữ kết quả và bước giải
var partOfStep5;
var rightStep2La;
var rightStep2InteLa;
var partOfStep5La;
var partCOfStep5La;

// Lấy các phần tử HTML từ trang để làm việc với chúng
var yDiffInput = document.querySelector(".solution__y-diff-input");
var pInput = document.querySelector(".solution__p-input");
var qInput = document.querySelector(".solution__q-input");
var solutionAns = document.querySelector(".solution__ans");
var step_0 = document.querySelector(".step-0");
var step_1 = document.querySelector(".step-1");
var step_2 = document.querySelector(".step-2");
var step_3 = document.querySelector(".step-3");
var step_4 = document.querySelector(".step-4");
var step_5 = document.querySelector(".step-5");

// Hàm giải phương trình
function solve(check = false) {
  // Xóa các bước giải cũ mỗi khi chạy lại
  step_0.innerHTML = "";
  step_1.innerHTML = "";
  step_2.innerHTML = "";
  step_3.innerHTML = "";
  step_4.innerHTML = "";
  step_5.innerHTML = "";

  // Lấy giá trị từ các input
  yDiffText = yDiffInput.value;
  pText = pInput.value;
  qText = qInput.value;

  // Kiểm tra và làm đơn giản biểu thức P(x) và Q(x) nếu cần
  if (yDiffText) {
    pText = nerdamer(simplify(`${pText}/(${yDiffText})`)).text();
    qText = nerdamer(simplify(`${qText}/(${yDiffText})`)).text();
    
    // Hiển thị bước giải ban đầu
    step_0.innerHTML += "Rút gọn hệ số y': ";
    step_0.innerHTML += `\\(y' \\ + \\ (${nerdamer.convertToLaTeX(pText)}) \\)`;
    step_0.innerHTML += `\\( \\cdot y \\ = \\ ${nerdamer.convertToLaTeX(qText)} \\)`;
  }

  // Chuyển đổi các biểu thức P(x) và Q(x) thành dạng LaTeX
  var pTextLa = nerdamer.convertToLaTeX(pText);
  var qTextLa = nerdamer.convertToLaTeX(qText);

  // Tính tích phân của P(x)
  var pInte = nerdamer(integrate(`${pText}`, 'x')).text();
  var eExpP = nerdamer(simplify(exp(`${pInte}`))).text();
  var eExpPLa = nerdamer(simplify(`${eExpP}`)).text();

  // Tính biểu thức bên phải sau khi nhân với e^∫P(x)dx
  var rightStep2 = nerdamer(simplify(`${qText}*${eExpP}`)).text();
  var rightStep2Inte = nerdamer(integrate(`${rightStep2}`, 'x')).text();

  // Kiểm tra xem tích phân có chứa hàm 'erf' không, nếu có thì bỏ qua
  if (rightStep2Inte.includes("erf")) {
    rightStep2Inte = "";
    partOfStep5 = "";
  } else {
    partOfStep5 = nerdamer(simplify(`${rightStep2Inte}/(exp(${pInte}))`)).text();
  }

  // Tính giá trị của C ở bước cuối cùng
  var partCOfStep5 = nerdamer(simplify(exp(`-${pInte}`))).text();
  var pInteLa = nerdamer.convertToLaTeX(pInte);

  // Nếu chưa kiểm tra bước giải, thực hiện việc sửa chữa dấu mũ đôi trong công thức
  if (!check) {
    rightStep2La = nerdamer.convertToLaTeX(rightStep2);
    rightStep2InteLa = nerdamer.convertToLaTeX(rightStep2Inte);
    partOfStep5La = nerdamer.convertToLaTeX(partOfStep5);
    partCOfStep5La = nerdamer.convertToLaTeX(partCOfStep5);
  } else {
    // Sửa chữa dấu mũ đôi nếu có
    rightStep2La = fixDoubleExponent(rightStep2La);
    rightStep2InteLa = fixDoubleExponent(rightStep2InteLa);
    partOfStep5La = fixDoubleExponent(partOfStep5La);
    partCOfStep5La = fixDoubleExponent(partCOfStep5La);
  }

  // Hiển thị các bước giải từng bước một
  step_1.innerHTML += "Lấy: ";
  step_1.innerHTML += `\\( e^{\\int P(x)dx} \\ = \\ \\)`;
  step_1.innerHTML += `\\( e^{\\int ${pTextLa} dx} \\)`;
  step_1.innerHTML += `\\( \\ = \\ ${eExpPLa} \\)`;

  step_2.innerHTML += "Nhân cả 2 vế cho ";
  step_2.innerHTML += "\\( e^{\\int P(x)dx}: \\ \\)";
  step_2.innerHTML += `\\( y' . ${eExpPLa} \\ + (${pTextLa} . ${eExpPLa}) . y \\ = \\ ${qTextLa}.${eExpPLa} \\)`;

  step_3.innerHTML += "Hay: ";
  step_3.innerHTML += "\\( \\frac{d}{dx} \\)";
  step_3.innerHTML += `\\( (y.${eExpPLa}) \\ = \\ )`;
  step_3.innerHTML += `\\( ${rightStep2La} \\)`;

  step_4.innerHTML += "Lấy tích phân 2 vế ta được: ";
  step_4.innerHTML += `\\( y.${eExpPLa} \\ = \\)`;
  step_4.innerHTML += `\\(  \\int ${rightStep2La} dx + C \\)`;
  step_4.innerHTML += `\\( ${(rightStep2Inte == "" || rightStep2 == "0") ? "" : `= ${rightStep2InteLa} + C`} \\)`;
  if (rightStep2 == "0") step_4.innerHTML += "\\( \\ = \\ C \\)";

  step_5.innerHTML += "Vậy nghiệm tổng quát của phương trình là: ";
  step_5.innerHTML += "\\( y \\ = \\ \\)";
  if (partOfStep5 == "") step_5.innerHTML += `\\( \\frac{\\int ${rightStep2La} dx}{${eExpPLa}} \\ + \\ \\)`;
  else step_5.innerHTML += `${partOfStep5La} \\ + \\ \\`;
  step_5.innerHTML += `${partCOfStep5La}.C \\`;

  // Kích hoạt MathJax để hiển thị các công thức toán học
  MathJax.typesetPromise([solutionAns]);

  // Kiểm tra xem có lỗi "Double exponent: use braces to clarify" và gọi lại hàm giải nếu cần
  if (solutionAns.innerHTML.includes("Double exponent: use braces to clarify") && !check) {
    check = true;
    solve(check);
  }
}

// Hàm sửa chữa dấu mũ đôi trong biểu thức
function fixDoubleExponent(expression) {
  const test = /{e}\^{([^}]+}\s*\^[^\{][0-9]+)/g;

  expression = expression.replaceAll(test, (match, p1) => `e^{{${p1}}}`);

  return expression;
}
</script>
</html>
