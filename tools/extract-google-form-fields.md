
# Extract Google Form Fields
This is a simple tool that helps you extract your Google Form field names.

1. Open your Google Form (in preview mode, not in edit mode)
2. Select all using cmd/ctrl+A and copy using cmd/ctrl+C
3. Paste here using cmd/ctrl+V

<section class="paste">
  Copy-Paste your Google Form here...
  <div id="error"></div>
</section>

<section id="results">
  <pre><code id="list"></code></pre>
</section>

*Note: All processing is performed in the client (your form data is not sent anywhere). Take a look at the page source to see the implementation. Tested with Chrome, Firefox, Opera, and Safari.*

<style>
  .paste {
    border: dashed 1px black;
    padding: 15px;
    text-align: center;
    border-radius: 8px;
    margin-bottom: 15px;
  }

  #error {
    color: red;
  }

  input[type=text] {
    width: 100%;
    border: solid 2px #ccc;
    border-radius: 8px;
    padding: 8px;
  }

  #results {
    display: none;
  }

  pre {
    border-radius: 8px;
    background: #eee;
    padding: 15px 20px;
    font-size: 1em;
  }
</style>

<script>
  (function () {

    function decodeHtml(html) {
      var txt = document.createElement('textarea');
      txt.innerHTML = html;
      return txt.value;
    }

    function load(data) {
      
      const token = 'data-params="%.@.';

      const questions = [];

      let pos;
      while ((pos = data.indexOf(token, pos + 1)) > -1) {
        const start = pos + token.length;
        const end = data.indexOf('"', start);
        const json = decodeHtml(data.slice(start, end));
        const params = JSON.parse('[' + json);
        const id = params[0][4][0][0];
        const label = params[0][1];
        if (!questions.find(e => e.id == id)) {
          questions.push({
            label: params[0][1],
            id: params[0][4][0][0],
          });
        }
      }

      if (questions.length == 0) {
        results.style.display = 'none';
        error.innerText = 'Are you sure you pasted your Google Form? Open your Google Form preview, press Ctrl/Cmd+A to select the entire page, then press Cmd+C to copy the page. Press Cmd+V here.';
      } else {
        error.innerText = '';
        results.style.display = 'block';
        list.innerText = questions.map(q => `entry.${q.id}: ${q.label}`).join('\n');
      }
    }

    document.addEventListener('paste', async (ev) => {
      ev = (ev.originalEvent || ev);
      if (ev.target.nodeName === 'INPUT') {
        return;
      }
      ev.preventDefault();
      let data = ev.clipboardData.getData('text/html');
      if (!data) {
        data = ev.clipboardData.getData('text/plain');
      }
      if (data) {
        load(data);
      }
    });

  })();
</script>
