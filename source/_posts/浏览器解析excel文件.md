title: 浏览器解析excel文件
author: Billy Yin
tags: []
categories: []
date: 2022-04-07 16:57:00
---
使用 [fileReader](https://developer.mozilla.org/zh-CN/docs/Web/API/FileReader) 可以搞定

但实际工作中倾向于寻找库来解决， 如
https://github.com/SheetJS/sheetjs

一个小demo
```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<body>
    <script src="./xlsx.core.min.js"></script>
    <input type="file" id="excel-file">
    <script>
        const change = function (e) {
            var files = e.target.files;

            var fileReader = new FileReader();
            fileReader.onload = function (ev) {
                try {
                    var data = ev.target.result,
                        workbook = XLSX.read(data, {
                            type: 'binary'
                        }), // 以二进制流方式读取得到整份excel表格对象
                        persons = []; // 存储获取到的数据
                } catch (e) {
                    console.log('文件类型不正确');
                    return;
                }

                // 表格的表格范围，可用于判断表头是否数量是否正确
                var fromTo = '';
                console.log(workbook)
                // 遍历每张表读取
                for (var sheet in workbook.Sheets) {
                    if (workbook.Sheets.hasOwnProperty(sheet)) {
                        fromTo = workbook.Sheets[sheet]['!ref'];
                        persons = persons.concat(XLSX.utils.sheet_to_json(workbook.Sheets[sheet]));
                        break; // 如果只取第一张表，就取消注释这行
                    }
                }

                console.log(persons);
            };

            // 以二进制方式打开文件
            fileReader.readAsBinaryString(files[0]);
        };
        const excel = document.querySelector('#excel-file')
        excel.addEventListener('change', change)
    </script>


</body>

</html>
```