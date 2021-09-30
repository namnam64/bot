var access_token =
  "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx";
var ss = SpreadsheetApp.openByUrl(
  "https://docs.google.com/spreadsheets/d/xxxxxxxxxxxxxxxxxxxxxxx/edit"
);
var sheet = ss.getSheets()[0];
var sheet_2 = ss.getSheets()[1];

function reply(token, replyText) {
  var url = "https://api.line.me/v2/bot/message/reply";

  var headers = {
    "Content-Type": "application/json; charset=UTF-8",
    Authorization: "Bearer " + access_token
  };
  var postData = {
    replyToken: token,
    messages: [
      {
        type: "text",
        text: replyText
      }
    ]
  };
  var options = {
    method: "POST",
    headers: headers,
    payload: JSON.stringify(postData)
  };
  return UrlFetchApp.fetch(url, options);
}

function doPost(e) {
  var json = JSON.parse(e.postData.contents).events[0];
  var j_mes = json.message.text;
  var replyToken = json.replyToken;

  if (j_mes.match(/@add/) !== null) {
    message = j_mes.substr(5);

    add_message = add_(message);
    reply(replyToken, add_message);
  } else if (j_mes.match(/@list/) !== null) {
    list_message = list_();
    reply(replyToken, list_message);
  } else if (j_mes.match(/@total/) !== null) {
    message = j_mes.substr(7);

    total_message = total_(message);
    reply(replyToken, total_message);
  } else if (j_mes.match(/@cut/) !== null) {
    message = j_mes.substr(5);

    cut_message = cut_(message);
    reply(replyToken, cut_message);
  } else if (j_mes.match(/@คู่มือ/) !== null) {
    bot_message = robot_();
    reply(replyToken, bot_message);
  } else {
    return;
  }
}

function add_(mes) {
  var add_text = mes + "\nเพิ่มรายการเรียบร้อย!!";
  var split_mes = mes.split("\n");

  sheet.appendRow(split_mes);
  sheet_2.appendRow(split_mes);

  return add_text;
}

function list_() {
  var list_text = "";
  var values = [];

  var data = sheet.getRange(1, 1, 40, 2).getValues();
  for (let i = 0; i < 40; i++) {
    if (typeof data[i][0] === "string" || typeof data[i][0] === "number") {
      var data_1 = data[i][0];
    } else {
      var data_1 = "";
    }
    if (typeof data[i][1] === "string" || typeof data[i][1] === "number") {
      var data_2 = data[i][1];
    } else {
      var data_2 = "";
    }
    var judge = data_1 === "" && data_2 === "";
    if (judge === false) {
      values.push([data_1, data_2]);
    }
  }

  var datava = values.length;

  if (datava >= 2) {
    for (let i = 1; i <= datava - 1; i++) {
      list_text =
        list_text +
        i.toString() +
        "  " +
        values[i - 1][0] +
        "  " +
        values[i - 1][1] +
        "\n";
    }
  }
  list_text =
    list_text +
    datava.toString() +
    "  " +
    values[datava - 1][0] +
    "  " +
    values[datava - 1][1];
  return list_text;
}

function total_(mes) {
  var amount = parseInt(mes);
  var sum = 0;
  var breakr = 0;

  if (isNaN(amount)) {
    amount = 2;
  }

  for (let i = 1; ; i++) {
    var values = sheet.getRange(i, amount).getValues();
    var values_num = parseInt(values[0]);

    if (isNaN(values_num) === false) {
      sum = sum + values_num;
    } else {
      breakr = breakr + 1;

      if (breakr > 5) {
        break;
      }
    }
  }
  var total_text = sum.toString() + " บาท";
  return total_text;
}

function cut_(mes) {
  var amount = parseInt(mes);

  if (isNaN(amount) === false) {
    var range = sheet.getRange(amount, 1, 1, 2);
    var values = range.getValues();

    var cut_text = "แถวที่ " + mes + "ได้ทำการยกเลิก\nเรียบร้อยแล้ว!!";

    var archive = new Array(2);
    for (let i = 0; i < 2; i++) {
      archive[i] = values[0][i];
    }
    sheet_2.appendRow(archive);
    sheet_2.appendRow(["👆 รายการหักออก", "👆 รายการหักออก"]);

    range.deleteCells(SpreadsheetApp.Dimension.ROWS);
  } else {
    var cut_text = "ลำดับที่ยกเลิก error : \nโปรดใส่รายการที่จะยกเลิกอีกครั้ง";
  }
  return cut_text;
}

function robot_() {
  var robot_mes =
    "👇 คู่มือในการใช้งาน 👇\n\n@add\nข้อความ\nจำนวนตัวเลข(number)\n\n@list  โชว์รายการไม่เกิน 40 บรรทัด\n\n@total\n(จำนวนเงิน, default -> 2)\n\n@cut\nตัวเลขของแถว\n\nหากสงสัยสอบถามเจ้าของร้าน!!";
  return robot_mes;
}
