# 점심 메뉴 추천 🍽️🍕🍜

이 프로젝트는 **점심 메뉴 추천 웹 앱** 입니다.  
사용자가 입력한 메시지를 바탕으로, 20년 경력의 메뉴 추천 전문가가 되어  
최고의 점심 메뉴를 추천해주는 서비스입니다! 🚀🤖

## 기능 소개 ✨

- **모델 선택** 🎯  
  - 두 가지 추천 모델 중 선택 가능!  
    - **gemini-1.5-flash** 🤝  
    - **deepseek-r1** 👀

- **실시간 데이터 업데이트** 🔄  
  - **Proxy**를 활용해 데이터 변경 시 화면이 자동으로 갱신됩니다.
  
- **로컬 스토리지 저장** 💾  
  - 추천 결과는 브라우저의 `localStorage`에 저장되어,  
    새로고침 후에도 데이터를 유지할 수 있습니다.

- **데이터 관리 기능** 🗑️  
  - 개별 추천 결과마다 **삭제** 버튼 제공  
  - 상단의 **"저장된 데이터 리셋"** 버튼으로 전체 데이터를 초기화!

- **API 연동** 🌐  
  - 입력 메시지를 바탕으로 두 개의 다른 API에 POST 요청을 보내,  
    각 모델에 따른 추천 메뉴를 받아옵니다.

## 사용 방법 🚀

1. **프로젝트 실행하기**  
   - 이 프로젝트는 HTML, CSS, JavaScript(ES6+)로 만들어졌습니다.  
   - `index.html` 파일을 브라우저에서 열어 실행해보세요! 🌟

2. **메뉴 추천 요청하기**  
   - 상단의 **"저장된 데이터 리셋"** 버튼으로 기존 데이터를 초기화할 수 있습니다. 🔄  
   - 폼에서 원하는 **모델**을 선택하고, 텍스트 영역에 메시지를 작성한 후 **등록** 버튼을 클릭하세요.  
   - 입력된 메시지를 바탕으로 API 요청을 보내고, 추천 결과가 화면에 나타납니다. 🎉

3. **데이터 관리**  
   - 각 추천 결과 옆에 있는 **"삭제"** 버튼을 클릭하여 개별 데이터를 삭제할 수 있습니다. ❌  
   - 모든 데이터는 브라우저의 `localStorage`에 저장되므로,  
     언제든지 다시 불러와 확인할 수 있습니다. 💡

## 코드 구조 🛠️

프로젝트의 핵심 코드는 `index.html` 내에 작성되어 있으며, 주요 부분은 아래와 같습니다:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>점심 메뉴 추천</title>
  </head>
  <body>
    <header>
      <button id="resetButton">저장된 데이터 리셋</button>
    </header>
    <form id="controller">
      <label>
        모델 :
        <select name="modelOption">
          <option value="1">gemini-1.5-flash</option>
          <option value="2">deepseek-r1</option>
        </select>
      </label>
      <textarea name="textData"></textarea>
      <button>등록</button>
    </form>
    <div id="container"></div>

    <script>
      // 페이지 로드 시 실행되는 함수
      const _ = () => {
        // HTML 요소들을 JavaScript로 가져오기 🎯
        const container = document.querySelector("#container");
        const form = document.querySelector("#controller");
        const resetButton = document.querySelector("#resetButton");

        // "저장된 데이터 리셋" 버튼 클릭 시 데이터를 비우기
        resetButton.addEventListener("click", () => (data.length = 0));

        // 데이터 저장을 위한 Proxy 객체 🛡️
        const data = new Proxy([], {
          set(target, property, value) {
            target[property] = value; // 배열에 값 저장
            updateContainer();      // 화면 업데이트
            updateStorage(target);  // localStorage에 데이터 저장
            return true;
          },
        });

        // 페이지 로드 시 localStorage에 저장된 데이터를 불러옴
        function onMounted() {
          data.push(...(JSON.parse(localStorage.getItem("myData")) ?? []));
        }
        onMounted();

        // 화면을 최신 상태로 업데이트하는 함수 🔄
        function updateContainer() {
          container.innerHTML = ""; // 기존 화면 비우기
          for (const d of data) {
            const tmp = document.createElement("div");
            tmp.textContent = d.text;

            // "삭제" 버튼 생성 및 이벤트 등록
            const deleteButton = document.createElement("button");
            deleteButton.textContent = "삭제";
            deleteButton.addEventListener("click", () => {
              const filtered = data.filter((value) => value.id !== d.id);
              data.length = 0;
              data.push(...filtered);
            });

            // 답변이 있을 경우, 답변 표시
            if (d.reply) {
              const box = document.createElement("div");
              const reply = document.createElement("code");
              box.appendChild(reply);
              box.style.padding = "12px";
              box.style.margin = "4px";
              box.style.backgroundColor = "beige";
              box.style.border = "1px solid black";
              reply.textContent = d.reply;
              tmp.appendChild(box);
            }

            tmp.appendChild(deleteButton);
            container.appendChild(tmp);
          }
        }

        // 데이터를 localStorage에 저장하는 함수 💾
        function updateStorage(target) {
          localStorage.setItem("myData", JSON.stringify(target));
        }

        // 폼 제출 시 실행되는 함수 (API 요청 포함) 📡
        async function handleForm(event) {
          event.preventDefault();
          const formData = new FormData(form);
          const text = formData.get("textData");
          let reply;

          switch (formData.get("modelOption")) {
            case "1":
              reply = `👬 Gemini : ${await makeReply(text)}`;
              break;
            case "2":
              reply = `👀 DeepSeek : ${await makeReply2(text)}`;
              break;
            default:
              alert("비정상적인 접근!");
              throw new Error("알 수 없는 에러!");
          }

          const displayData = {
            id: Date.now(),
            text,
            reply,
          };
          data.push(displayData);
        }

        // 첫 번째 모델 API 호출 함수
        async function makeReply(text) {
          const url = "https://spotted-famous-seaplane.glitch.me/1";
          const response = await fetch(url, {
            method: "POST",
            body: JSON.stringify({
              text: `너는 20년 경력의 메뉴 추천 전문가야. {${text}}의 메시지를 바탕으로, 점심 메뉴를 추천해주고, 50자 이내에 평문으로 작성해줘.`,
            }),
            headers: { "Content-Type": "application/json" },
          });
          const json = await response.json();
          return json.reply;
        }

        // 두 번째 모델 API 호출 함수
        async function makeReply2(content) {
          const url = "https://spotted-famous-seaplane.glitch.me/2";
          const response = await fetch(url, {
            method: "POST",
            body: JSON.stringify({
              text: `너는 20년 경력의 메뉴 추천 전문가야. {${content}}의 메시지를 바탕으로, 점심 메뉴를 추천해주고, 50자 이내에 평문으로 작성해줘. 결과는 한글로 작성해줘.`,
            }),
            headers: { "Content-Type": "application/json" },
          });
          const json = await response.json();
          return json.reply;
        }

        // 폼 제출 이벤트 등록
        form.addEventListener("submit", handleForm);
      };

      window.onload = _;
    </script>
  </body>
</html>
```
## 기술 스택 💻
- HTML5 📝
- CSS3 🎨
- JavaScript (ES6+) 🚀
- Fetch API 🌐
- LocalStorage 💾
## 데모 & 배포 🎬
이 프로젝트는 Glitch와 같은 클라우드 플랫폼에서 호스팅하여,
실시간으로 작동하는 웹 앱을 직접 체험할 수 있습니다. 🔥

## 기여 방법 🤝
이 프로젝트를 포크하세요! 🍴
새로운 브랜치에서 기능 추가 및 버그 수정을 진행하세요. 🛠️
변경 사항을 커밋하고, 풀 리퀘스트(PR)를 생성해 주세요. 🚀
