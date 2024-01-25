<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            display: flex;
            align-items: center;
            justify-content: center;
            min-height: 100vh;
            margin: 0;
            background-color: #f0f0f0;
            overflow-y: auto;
        }

        #container {
            border: 2px solid #3498db;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
            background-color: #fff;
            max-width: 600px;
            width: 100%;
        }

        table {
            width: 100%;
            border-collapse: collapse;
            margin-top: auto;
            margin-bottom: 20px;
        }

        table,
        th,
        td {
            border: 1px solid #3498db;
        }

        th,
        td {
            padding: 15px;
            text-align: center;
        }

        th {
            background-color: #3498db;
            color: #fff;
        }

        tr:nth-child(even) {
            background-color: #f2f2f2;
        }

        input {
            width: calc(100% - 10px);
            box-sizing: border-box;
            text-align: center;
            padding: 10px;
            margin-bottom: 10px;
            border: 1px solid #3498db;
            border-radius: 5px;
        }

        button {
            padding: 15px;
            background-color: #3498db;
            color: #fff;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            width: 100%;
            transition: background-color 0.3s;
        }

        button:hover {
            background-color: #258cd1;
        }

        .deleteBtn {
            background-color: #e74c3c;
        }

        .deleteBtn:hover {
            background-color: #c0392b;
        }

        #report {
            margin-top: 20px;
            padding: 15px;
            border: 1px solid #3498db;
            border-radius: 5px;
            background-color: #fff;
        }

        p {
            margin: 0;
            font-size: 18px;
            color: #333;
        }
    </style>
</head>

<body>

    <div id="container">
        <table id="myTable">
            <thead>
                <tr>
                    <th>Bill Number</th>
                    <th>Amount 1</th>
                    <th>Amount 2</th>
                    <th>Exceed Amount 1</th>
                    <th>Exceed Amount 2</th>
                    <th>Action</th>
                </tr>
            </thead>
            <tbody></tbody>
        </table>

        <input type="text" placeholder="Bill Number" maxlength="4" id="billNumber" oninput="validateBillNumber(this)" onkeyup="moveToNextBox(event, this, 2)">
        <input type="text" placeholder="Amount 1" id="amount1" oninput="validateAmount(this)" onkeyup="moveToNextBox(event, this, 3)">
        <input type="text" placeholder="Amount 2" id="amount2" oninput="validateAmount(this)" onkeyup="moveToNextBox(event, this, 'submitBtn')">
        <button id="submitBtn" onclick="submitForm()">Submit</button>

        <button onclick="generateReport()">Generate Report</button>

        <button onclick="subtractExceedAmount()">Done</button>

        <button onclick="clearData()">Clear Data</button>

        <div id="report"></div>
    </div>

    <script>
        // Function to validate bill number in Box 1
        function validateBillNumber(input) {
            input.value = input.value.replace(/[^0-9+]/g, '');
            if (input.value.length > 4) {
                input.value = input.value.slice(0, 4);
            }
        }

        // Function to validate amount in Boxes 2 and 3
        function validateAmount(input) {
            input.value = input.value.replace(/[^0-9.]/g, '');
            input.value = input.value.replace(/^0+/, '');
            var decimalIndex = input.value.indexOf('.');
            if (decimalIndex !== -1 && input.value.length - decimalIndex > 3) {
                input.value = input.value.slice(0, decimalIndex + 3);
            }
        }

        // Function to move to the next input box on Enter key release
        function moveToNextBox(event, currentInput, nextBox) {
            if (event.key === 'Enter') {
                event.preventDefault();

                var inputs = document.querySelectorAll('input');
                var currentIndex = Array.from(inputs).indexOf(currentInput);
                var nextInput;

                if (nextBox === 'submitBtn') {
                    submitForm();
                    return;
                } else {
                    nextInput = inputs[currentIndex + 1];
                }

                if (nextInput) {
                    nextInput.focus();
                } else {
                    document.getElementById('submitBtn').focus();
                }
            }
        }

        // Function to submit the form and add a new row or update amounts in the table
        function submitForm() {
            var billNumber = document.getElementById('billNumber').value;
            var amount1 = parseFloat(document.getElementById('amount1').value) || 0;
            var amount2 = parseFloat(document.getElementById('amount2').value) || 0;

            if (billNumber && (amount1 || amount2)) {
                var tableBody = document.getElementById('myTable').getElementsByTagName('tbody')[0];
                var rows = tableBody.getElementsByTagName('tr');
                var existingRow = null;

                for (var i = 0; i < rows.length; i++) {
                    var cells = rows[i].getElementsByTagName('td');
                    if (cells.length > 0 && cells[0].innerText === billNumber) {
                        existingRow = rows[i];
                        break;
                    }
                }

                if (existingRow) {
                    var existingAmount1 = parseFloat(existingRow.cells[1].innerText) || 0;
                    var existingAmount2 = parseFloat(existingRow.cells[2].innerText) || 0;

                    existingRow.cells[1].innerText = (existingAmount1 + amount1).toFixed(2);
                    existingRow.cells[2].innerText = (existingAmount2 + amount2).toFixed(2);
                } else {
                    var newRow = document.createElement('tr');
                    newRow.innerHTML = `<td>${billNumber}</td><td>${amount1.toFixed(2)}</td><td>${amount2.toFixed(2)}</td><td>0.00</td><td>0.00</td><td><button class="deleteBtn" onclick="deleteRow(this.parentNode.parentNode)">Delete</button></td>`;
                    tableBody.appendChild(newRow);
                }

                document.getElementById('billNumber').value = '';
                document.getElementById('amount1').value = '';
                document.getElementById('amount2').value = '';

                setTimeout(function () {
                    document.getElementById('billNumber').focus();
                }, 10);
            } else {
                alert('Please fill in at least one amount and the bill number before submitting.');
            }
        }

        // Function to generate and display a report
        function generateReport() {
            var thresholdAmount1 = parseFloat(prompt("Enter the threshold amount for Amount 1:"));

            if (isNaN(thresholdAmount1)) {
                alert("Invalid threshold amount for Amount 1. Please enter a valid number.");
                return;
            }

            var thresholdAmount2 = parseFloat(prompt("Enter the threshold amount for Amount 2:"));

            if (isNaN(thresholdAmount2)) {
                alert("Invalid threshold amount for Amount 2. Please enter a valid number.");
                return;
            }

            var tableBody = document.getElementById('myTable').getElementsByTagName('tbody')[0];
            var rows = tableBody.getElementsByTagName('tr');

            var exceededBills = [];

            for (var i = 0; i < rows.length; i++) {
                var cells = rows[i].getElementsByTagName('td');
                if (cells.length >= 3) {
                    var billNumber = cells[0].innerText;
                    var amount1 = parseFloat(cells[1].innerText) || 0;
                    var amount2 = parseFloat(cells[2].innerText) || 0;

                    var exceedAmount1 = Math.max(0, amount1 - thresholdAmount1);
                    var exceedAmount2 = Math.max(0, amount2 - thresholdAmount2);

                    cells[3].innerText = exceedAmount1.toFixed(2);
                    cells[4].innerText = exceedAmount2.toFixed(2);

                    if (exceedAmount1 > 0 || exceedAmount2 > 0) {
                        exceededBills.push({ billNumber, exceedAmount1, exceedAmount2 });
                    }
                }
            }

            var reportDiv = document.getElementById('report');

            if (exceededBills.length > 0) {
                var reportContent = `<p>Bills with amounts exceeding the specified thresholds:</p><ul>`;
                exceededBills.forEach(bill => {
                    reportContent += `<li>Bill Number: ${bill.billNumber}, 
                                      Exceed Amount 1: ${bill.exceedAmount1.toFixed(2)}, 
                                      Exceed Amount 2: ${bill.exceedAmount2.toFixed(2)}</li>`;
                });
                reportContent += `</ul>`;
                reportDiv.innerHTML = reportContent;
            } else {
                reportDiv.innerHTML = `<p>No bills have amounts exceeding the specified thresholds.</p>`;
            }
        }

        // Function to subtract exceed amount
        function subtractExceedAmount() {
            var tableBody = document.getElementById('myTable').getElementsByTagName('tbody')[0];
            var rows = tableBody.getElementsByTagName('tr');

            for (var i = 0; i < rows.length; i++) {
                var cells = rows[i].getElementsByTagName('td');
                if (cells.length >= 6) {
                    var exceedAmount1 = parseFloat(cells[3].innerText) || 0;
                    var exceedAmount2 = parseFloat(cells[4].innerText) || 0;

                    var amount1 = parseFloat(cells[1].innerText) || 0;
                    var amount2 = parseFloat(cells[2].innerText) || 0;

                    cells[1].innerText = (amount1 - exceedAmount1).toFixed(2);
                    cells[2].innerText = (amount2 - exceedAmount2).toFixed(2);
                    cells[3].innerText = '0.00';
                    cells[4].innerText = '0.00';
                }
            }
        }

        // Function to delete a row
        function deleteRow(row) {
            var table = document.getElementById('myTable');
            var rowIndex = row.rowIndex;
            table.deleteRow(rowIndex);
        }

        // Function to clear data
        function clearData() {
            localStorage.removeItem('tableData');
            loadTableData();
        }

        // Function to save table data to localStorage
        function saveTableData() {
            var tableBody = document.getElementById('myTable').getElementsByTagName('tbody')[0];
            var rows = tableBody.getElementsByTagName('tr');
            var data = [];

            for (var i = 0; i < rows.length; i++) {
                var cells = rows[i].getElementsByTagName('td');
                if (cells.length >= 5) {
                    data.push({
                        billNumber: cells[0].innerText,
                        amount1: cells[1].innerText,
                        amount2: cells[2].innerText,
                        exceedAmount1: cells[3].innerText,
                        exceedAmount2: cells[4].innerText,
                    });
                }
            }

            localStorage.setItem('tableData', JSON.stringify(data));
        }

        // Function to load table data from localStorage
        function loadTableData() {
            var tableBody = document.getElementById('myTable').getElementsByTagName('tbody')[0];
            tableBody.innerHTML = '';

            var data = localStorage.getItem('tableData');

            if (data) {
                data = JSON.parse(data);

                for (var i = 0; i < data.length; i++) {
                    var newRow = document.createElement('tr');
                    newRow.innerHTML = `<td>${data[i].billNumber}</td>
                                       <td>${data[i].amount1}</td>
                                       <td>${data[i].amount2}</td>
                                       <td>${data[i].exceedAmount1}</td>
                                       <td>${data[i].exceedAmount2}</td>
                                       <td><button class="deleteBtn" onclick="deleteRow(this.parentNode.parentNode)">Delete</button></td>`;
                    tableBody.appendChild(newRow);
                }
            }
        }

        // Load table data on page load
        window.onload = loadTableData;

        // Save table data on page unload
        window.onbeforeunload = saveTableData;
    </script>
</body>

</html>
