<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
  </div>
</template>

<script>
// import axios from "axios";
import createReport from "docx-templates";
export default {
  name: "HelloWorld",
  setup() {
    const data1 = {
      project: {
        name: "docx-templates",
        details: { year: "2016" },
        img: "/1.png",
        people: [
          { name: "John", since: 2015 },
          { name: "Robert", since: 2010 },
        ],
      },
    };

    const promiseVal = () => {
      return new Promise((resolve) => {
        fetch("/1.docx").then((res) => {
          resolve(res.blob());
        });
      });
    };
    promiseVal().then((e) => {
      console.log(e, "0");
      onTemplateChosen(e);
    });

    const readFileIntoArrayBuffer = async (fd) =>
      new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.onerror = reject;
        reader.onload = () => {
          resolve(reader.result);
        };
        reader.readAsArrayBuffer(fd);
      });
    const onTemplateChosen = async (e) => {
      const template = await readFileIntoArrayBuffer(e);
      console.log(template, "1");
      const report = await createReport({
        template,
        data: data1,
        additionalJsContext: {
          tile: async (z, x, y, size = 1) => {
            const resp = await fetch(
              `http://tile.stamen.com/toner/${z}/${x}/${y}.png`
            );
            const buffer = resp.arrayBuffer
              ? await resp.arrayBuffer()
              : await resp.buffer();
            return {
              width: size,
              height: size,
              data: buffer,
              extension: ".png",
            };
          }
        },
      });
      saveDataToFile(
        report,
        "report.docx",
        "application/vnd.openxmlformats-officedocument.wordprocessingml.document"
      );
    };
    // axios({
    //   method: "get",
    //   url: "/1.docx",
    //   responseType: "arraybuffer",
    // }).then(function(res) {
    //   const blob = res.data;
    //   console.log(res, "blob");
    //   let report = async () =>
    //     await createReport({
    //       blob,
    //       data: data1,
    //     });
    //   setTimeout(() => {
    //     console.log(report, "report");
    //     saveDataToFile(
    //       blob,
    //       "report.docx",
    //       "application/vnd.openxmlformats-officedocument.wordprocessingml.document"
    //     );
    //   }, 3000);
    // });

    const saveDataToFile = (data, fileName, mimeType) => {
      const blob = new Blob([data], { type: mimeType });
      const url = window.URL.createObjectURL(blob);
      downloadURL(url, fileName, mimeType);
      setTimeout(() => {
        window.URL.revokeObjectURL(url);
      }, 1000);
    };
    const downloadURL = (data, fileName) => {
      const a = document.createElement("a");
      a.href = data;
      a.download = fileName;
      document.body.appendChild(a);
      a.style = "display: none";
      a.click();
      a.remove();
    };
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
