<!DOCTYPE html>
<html lang="vi">
<head>
<meta charset="UTF-8" />
<title>Vòng Quay May Mắn - Dừng ở người thứ 3</title>
<style>
  body {
    font-family: Arial, sans-serif;
    text-align: center;
    margin: 20px;
  }

  /* Vòng quay */
  .wheel {
    width: 300px;
    height: 300px;
    border: 10px solid #333;
    border-radius: 50%;
    position: relative;
    margin: 100px auto 30px;
    overflow: hidden;
    user-select: none;
    background: #eee;
    transition: transform 4s ease-out;
  }

  /* Các phân đoạn trên vòng quay */
  .segment {
    position: absolute;
    width: 50%;
    height: 50%;
    background: #3498db;
    color: white;
    font-weight: bold;
    display: flex;
    justify-content: center;
    align-items: center;
    top: 50%;
    left: 50%;
    transform-origin: 0% 0%;
    border: 1px solid #fff;
    box-sizing: border-box;
    text-shadow: 1px 1px 2px black;
    font-size: 14px;
  }

  #pointer {
  width: 0;
  height: 0;
  border-left: 20px solid transparent;
  border-right: 20px solid transparent;
  border-top: 30px solid red;  /* mũi tên chỉ xuống */
  position: absolute;
  top:  60px;   /* nằm trên vòng quay */
  left: 50%;
  transform: translateX(-50%);
  z-index: 10;
}


  /* Kết quả */
  #result {
    font-size: 24px;
    margin-top: 20px;
    font-weight: bold;
    color: green;
    min-height: 30px;
  }

  /* Nút */
  button {
    padding: 10px 15px;
    margin: 10px 5px;
    font-size: 16px;
    cursor: pointer;
  }

  /* Bảng danh sách */
  table {
    margin: 0 auto;
    margin-top: 20px;
    border-collapse: collapse;
    width: 350px;
  }

  td, th {
    padding: 6px 12px;
    border: 1px solid #ccc;
  }

  input[type="text"] {
    width: 100%;
    box-sizing: border-box;
  }

  .delete-btn {
    background-color: crimson;
    color: white;
    border: none;
    padding: 6px 10px;
    cursor: pointer;
    font-weight: bold;
  }

  .delete-btn:hover {
    background-color: darkred;
  }
</style>
</head>
<body>

<h1>🎡 Vòng Quay May Mắn</h1>

<div id="pointer"></div>

<div class="wheel" id="wheel"></div>

<button onclick="spin()">🎯 Quay</button>
<button onclick="addNameRow()">➕ Thêm tên</button>

<div id="result"></div>

<h3>Danh sách người chơi</h3>
<table id="nameTable">
  <tr>
    <th>STT</th>
    <th>Tên</th>
    <th>Thao tác</th>
  </tr>
</table>

<script>
  let nameList = [];
  const wheel = document.getElementById("wheel");
  const nameTable = document.getElementById("nameTable");
  const pointer = document.getElementById("pointer");
  let rotation = 0;

  function addNameRow() {
    nameList.push("");
    renderNameTable();
    drawWheel();
  }

  function deleteName(index) {
    nameList.splice(index, 1);
    renderNameTable();
    drawWheel();
  }

  function renderNameTable() {
    nameTable.innerHTML = `
      <tr>
        <th>STT</th>
        <th>Tên</th>
        <th>Thao tác</th>
      </tr>`;

    nameList.forEach((name, i) => {
      const row = document.createElement("tr");

      const sttCell = document.createElement("td");
      sttCell.innerText = i + 1;

      const nameCell = document.createElement("td");
      const input = document.createElement("input");
      input.type = "text";
      input.value = name;
      input.placeholder = "Nhập tên";
      input.oninput = () => {
        nameList[i] = input.value;
        drawWheel();
      };
      nameCell.appendChild(input);

      const actionCell = document.createElement("td");
      const delBtn = document.createElement("button");
      delBtn.innerText = "🗑 Xóa";
      delBtn.className = "delete-btn";
      delBtn.onclick = () => deleteName(i);
      actionCell.appendChild(delBtn);

      row.appendChild(sttCell);
      row.appendChild(nameCell);
      row.appendChild(actionCell);

      nameTable.appendChild(row);
    });
  }

  function drawWheel() {
    wheel.innerHTML = "";
    const count = nameList.length;
    if (count === 0) return;

    const segmentAngle = 360 / count;

    for (let i = 0; i < count; i++) {
      const segment = document.createElement("div");
      segment.className = "segment";

      segment.style.transform = `
        rotate(${segmentAngle * i}deg)
        skewY(${90 - segmentAngle}deg)
      `;
      segment.style.background = i % 2 === 0 ? '#3498db' : '#9b59b6';

      const textDiv = document.createElement("div");
      textDiv.style.transform = `skewY(${segmentAngle - 90}deg) rotate(${segmentAngle / 2}deg)`;
      textDiv.style.width = "140%";
      textDiv.style.textAlign = "center";
      textDiv.style.fontWeight = "bold";
      textDiv.style.fontSize = "14px";
      textDiv.style.userSelect = "none";
      textDiv.style.color = "white";
      textDiv.innerText = nameList[i] || `#${i + 1}`;

      segment.appendChild(textDiv);
      wheel.appendChild(segment);
    }
  }

  function spin() {
    const count = nameList.length;
    if (count < 3) {
      alert("Cần ít nhất 3 người để trúng STT 3!");
      return;
    }

    const fixedIndex = 2; // Người thứ 3 (index 2)
    const segmentAngle = 360 / count;

    // Tính góc để quay sao cho người thứ 3 nằm dưới mũi tên (mũi tên ở 0 độ)
    // Vì vòng quay quay ngược chiều kim đồng hồ, nên ta lấy:
    // rotation góc = 360 * số vòng quay + 270 độ - góc trung tâm của phần tử thứ 3
    // (270 độ vì mũi tên nằm ở trên, tương ứng 270 độ theo hệ toạ độ rotate CSS)
    
    const spins = 5; // số vòng quay tròn trước khi dừng
    const targetAngle = spins * 360 + 270 - (fixedIndex * segmentAngle + segmentAngle / 2);

    rotation = (rotation + targetAngle) % 3600; // tránh số quá lớn

    wheel.style.transition = "transform 4s ease-out";
    wheel.style.transform = `rotate(${rotation}deg)`;

    // Hiển thị kết quả sau khi quay xong
    setTimeout(() => {
      const name = nameList[fixedIndex].trim() || "Chưa nhập tên";
      document.getElementById("result").innerText = `🎉 Người trúng: ${name}`;
    }, 4200);
  }

  // Khởi tạo 3 tên mẫu
  function initDemo() {
    nameList = ["Nguyễn Văn A", "Trần Thị B", "Lê Văn C"];
    renderNameTable();
    drawWheel();
  }

  initDemo();
</script>

</body>
</html>
