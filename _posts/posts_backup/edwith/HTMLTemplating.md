---
title: "HTMLTemplating"
date: 2019-04-18T00:43:40+09:00
archives: "2019"
tags: ["javascript", "edwith", "webprogramming"]
author: Joohan Lee
---

## HTML Templating

### 1. 개념

- 반복적인 HTML부분을 template로 만들어두고, 서버에서 온 데이터(주로JSON)을 결합해서, 화면에 추가하거나 삭제하는 작업
- JSON 형태의 데이터를 받을 것이고요.

### 2. HTML과 JSON 결합 예제

- HTML 코드

```html
<li>
    <h4>{title}</h4>
    <p>{content}</p>
    <div>{price}</div>
</li>
```

- JSON Data

```json
{
    title: "Test Product",
    content: "This is a description",
    price: 2000
}
```

- 결합된 HTML

```html
<li>
    <h4>Test Product</h4>
    <p>This is a description</p>
    <div>2000</div>
</li>
```

- Console에서 실행해보기

```javascript
var data = {
    title: "Test Product",
    content: "This is a description",
    price: 2000
};

var html = "<li><h4>{title}</h4><p>{content}</p><div>{price}</div></li>";
var resultHTML = html.replace("{title}", data.title)
				.replace("{content}", data.content)
				.replace("{price}", data.price);

// resultHTML: "<li><h4>Test Product</h4><p>This is a description</p><div>2000</div>"
```

- data가 array형태로 왔을 때 처리

```javascript
var data = [
    {
    	title: "Test Product1",
    	content: "This is a description",
    	price: 2000
	},
    {
        title: "Test Product2",
    	content: "This is a description",
    	price: 3000
	}
];

var resultHTML = "";

data.forEach(function (dataObject) {
var html = "<li><h4>{title}</h4><p>{content}</p><div>{price}</div></li>";
resultHTML += html.replace("{title}", dataObject.title)
				.replace("{content}", dataObject.content)
				.replace("{price}", dataObject.price);
});

// resultHTML: "<li><h4>Test Product1</h4><p>This is a description</p><div>2000</div></li><li><h4>Test Product2</h4><p>This is a description</p><div>3000</div></li>"
```

### 3. HTML Template의 보관

```javascript
var html = "<li><h4>{title}</h4><p>{content}</p><div>{price}</div></li>";
```

- 위의 HTML Template을 사용하기 위해선 어딘가에 저장해야함
- javascript코드 안에서 이런 정적인 데이터를 보관하는 건 좋지 않음.
- 몇 가지 방법이 있다.
  - 서버에서 file로 보관하고 Ajax로 요청해서 받아오기 (ex. a.template, b.template과 같이 여러개를 가지고 있다가 맞는 template을 보내줌)
  - HTML코드 안에 숨겨두기 - 간단한 것이라면 HTML 안에 숨겨둘 수가 있음
- 숨겨야 할 데이터가 많다면 별도 파일로 분리해서 Ajax로 가져오는 방법도 좋음
- 일단은 HTML에 보관하는 방식으로 진행 (현업에서도 아직 자주 쓰이는 방법)

### 4. Templating

- HTML 중 script 태그는 type이 javascript가 아니라면 렌더링하지 않고 무시한다.
- javascript에서 가져온 후 replace처리

```html
<html>
	<header>
        <style>
        	.content {
	        	border: 1px solid gray;
	        }
        </style>
    </header>
    <script id="template-list-item" type="text/template">
	  	<li>
			<h4>{title}</h4><p>{content}</p><div>{price}</div>
  		</li>
	</script>
	<script>
	    //Mock data
	    var data = {
	    	title: "Test Product",
	    	content: "This is a description",
	    	price: 2000
		};

	    var html = document.querySelector("template-list-item").innerHTML;
		var resultHTML = html.replace("{title}", data.title)
    						.replace("{content}", data.content)
							.replace("{price}", data.price);

    	document.querySelector(".content").innerHTML = resultHTML;
	</script>
</html>
```

### 5. Tab UI

- 위 버튼을 누르면 아래 컨텐츠가 바뀌는 html templating 예제
- 사전 조건: Ajax Call을 처리할 서버를 준비해야 한다.

tabui.html

```html
<html>
    <header>
        <style>
            h2 {
                text-align: center;
            }
            h2, h4 {
                margin: 0px;
            }
            .tab {
                width: 600px;
                margin: 0px auto;
            }
            .tabmenu {
                background-color: bisque;
            }
            .tabmenu > div {
                text-align: center;
                display: inline-block;
                width: 140px;
                height: 50px;
                line-height: 50px;
                cursor: pointer;
            }
            .content {
                padding: 5%;
                background-color: antiquewhite;
            }
        </style>
    </header>
    <body>
        <h2> TAB UI TEST</h2>

        <div class="tab">
            <div class="tabmenu">
                <div>tab1</div>
                <div>tab2</div>
                <div>tab3</div>
                <div>tab4</div>
            </div>
            <section class="content">
                <h4>Hello World!</h4>
                <p>random text.</p>
            </section>
        </div>
        <script>

            function makeTemplate(data, clickedName) {
                var html = document.getElementById("tabcontent").innerHTML;
                var resultHTML = "";

                for (var i = 0; i < data.length; i++) {
                    if (data[i].name === clickedName) {
                        resultHTML = html.replace("{name}", data[i].name)
                            .replace("{favorites}", data[i].favorites.join(" "));
                        break;
                    }
                }
                document.querySelector(".content").innerHTML = resultHTML;
        }

            function sendAjax(url, clickedName) {
                var oReq = new XMLHttpRequest();
                oReq.addEventListener("load", function() {
                    var data = JSON.parse(oReq.responseText);
                    makeTemplate(data, clickedName);
                });
                oReq.open("GET", url)
                oReq.send();
            }

            var tabmenu = document.querySelector(".tabmenu");
            tabmenu.addEventListener("click", function(evt) {
                sendAjax("./json.txt", evt.target.innerText);
            });
        </script>

        <script id="tabcontent" type="my-template">
            <h4>hello {name}</h4>
            <p>{favorites}</p>
        </script>
</body>
    </body>
</html>
```

json.txt

```json
[
    {
    	"name": "Tester1",
    	"favorites": ["golf1", "jogging1"]
    },
    {
    	"name": "Tester2",
    	"favorites": ["golf2", "jogging2"]
    },
    {
    	"name": "Tester3",
    	"favorites": ["golf3", "jogging3"]
    }
]
```



원본 강의: https://www.edwith.org/boostcourse-web
