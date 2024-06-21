# gemini_api
gemini api demo

function getWebviewContent_geminiapi(selectedText) {
  const apikey = ""; // replace with your api key
  return `<!DOCTYPE html>
      <html lang="en">
      <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Selected Code</title>
        <style>
          body, html {
            height: auto;
            margin: 0;

            display: flex;
            flex-direction: column;
          }
          .container {
            background-color: #313131;
            width: 100%;
            padding: 10px;
            border: 1px solid #313131;
            border-radius: 4px;
            color: #cccccc;
            font-family: var(--vscode-editor-font-family);
            font-size: var(--vscode-editor-font-size);
            font-weight: var(--vscode-editor-font-weight);
          }
          .codearea {
            min-height: 3lh;
            max-height: 200px;
            width: 100%;
            resize: none;
          }
          .additional {
            height: 50px;
            width: 100%;
            resize: none;
          }
          #submit_button {
            background-color: #0078d3;
            color: #ffffff;
            padding: 10px;
            width: 100%;
            display: flex;
            margin-top:4px;
            align-items: center;
            justify-content: center;
            border-radius: 4px;
            cursor: pointer;
          }
          #result_div {
            width: 100%;
            margin-top: 10px;
            min-height:5lh;
            overflow-y: auto;
            resize: none;
          }
        </style>
      </head>
      <body>
        <h3>Selected Code</h3>
        <textarea class="container codearea" id="codearea" contenteditable="true">${escapeHtml(
          selectedText
        )}</textarea>
        <h3>Enter Prompt</h3>
        <textarea class="container additional" id="additional_text" contenteditable="true"></textarea>
        <div id="submit_button">Submit</div>
        <textarea id="result_div" class="container"></textarea>
        
        <script>
        const vscode = acquireVsCodeApi();

        function autoresize(){
              const textarea = document.getElementById("result_div");
              textarea.style.height = "auto";
              textarea.style.height = textarea.scrollHeight + "px";
        }

        document.getElementById("additional_text").addEventListener("input",()=>{
          const additional_text= document.getElementById("additional_text");
          additional_text.style.height = "auto";
          additional_text.style.height = additional_text.scrollHeight + "px";
        })

        const codearea = document.getElementById("codearea");
        codearea.style.height = "auto";
        codearea.style.height = codearea.scrollHeight + "px";



        const apiUrl = "https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash-latest:generateContent?key=${apikey}";
  
          function apicall() {
              const submitButton = document.getElementById("submit_button");
              submitButton.textContent = "Loading...";
              document.getElementById("result_div").style.display = "block";
              const codearea = document.getElementById("codearea").value;
              const additional_text = document.getElementById("additional_text").value;
              const data = additional_text + codearea ;
              fetchInChunks(apiUrl, data, () => {
                  submitButton.textContent = "Submit";
              });
          }
  
          async function fetchInChunks(url, requestData, callback) {
              document.getElementById("result_div").innerHTML = "";
              try {
                  const response = await fetch(url, {
                      method: 'POST',
                      headers: {
                          'Content-Type': 'application/json'
                      },
                      body: JSON.stringify(
                        {"contents":[{"parts":[{"text":requestData}]}]}
                      ),
   
                      stream: true // Enable streaming
                  });
  
                  if (!response.body) {
                      throw new Error('No response body');
                  }
  
                  const reader = response.body.getReader();
                  const decoder = new TextDecoder('utf-8');
  
                  while (true) {
                      const { done, value } = await reader.read();
                      if (done) {
                          console.log('Stream complete');
                          break;
                      }
                      const text = decoder.decode(value, { stream: true });
                      console.log('Received data:', text);
                      const jsonResponse = JSON.parse(text);
                      const content = jsonResponse.candidates[0].content.parts[0].text;
                      console.log('Text only:', content);
                      document.getElementById("result_div").innerHTML += content;
                      autoresize()
                      window.scrollTo(0, document.body.scrollHeight);
                  }
              } catch (error) {
                  console.error('Error:', error);
                  document.getElementById("result_div").textContent = 'Error: ' + error.message;
              } finally {
                  callback();
              }
          }
  
          document.getElementById("submit_button").addEventListener("click", function () {
              apicall();
          });
        
        </script>
        
      </body>
      </html>`;
}
function escapeHtml(unsafe) {
  return unsafe
    .replace(/&/g, "&amp;")
    .replace(/</g, "&lt;")
    .replace(/>/g, "&gt;")
    .replace(/"/g, "&quot;")
    .replace(/'/g, "&#039;");
}

module.exports = getWebviewContent_geminiapi;
