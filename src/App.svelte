<script>
  export let name;

  import axios from "axios";

  var RAW_REPO_BASE_URL =
    "https://raw.githubusercontent.com/getify/You-Dont-Know-JS/master/";
  var REPO_FILES_URL =
    "https://api.github.com/repos/getify/You-Dont-Know-JS/contents/";

  const books = [
    { name: "up & going" },
    { name: "scope & closures" },
    { name: "this & object prototypes" },
    { name: "types & grammar" },
    { name: "async & performance" },
    { name: "es6 & beyond" }
  ];

  let selectedBookName = ""; // books[0];
  let selectedChapter = "README.md";

  var sortedMarkdownFiles = [];

  function getBookContents() {
    console.log("get");
    axios.get(REPO_FILES_URL).then(response => {
      console.log(response);
    });
  }

  function bookSelectionChanged(selectEvent, kp) {
    selectBook(selectEvent);
    requestBookFolderContentsAndReadmeFile();
  }

  function selectBook(selectEvent) {
    const selectedIndex = selectEvent.target.selectedIndex;
    selectedBookName = books[selectedIndex].name;
  }

  function requestBookFolderContentsAndReadmeFile() {
    const encodedBookName = encodeURIComponent(selectedBookName);

    if (!encodedBookName) {
      return;
    }
    const readmeUrl = buildFileRequestUrl(encodedBookName, selectedChapter);
    const readme = axios.get(readmeUrl);
    const folderContents = axios.get(REPO_FILES_URL + encodedBookName);

    axios
      .all([folderContents, readme])
      // TODO remove readme, because md file sorting is not done by pasring readme listings
      // rather fixed table of contents is pretty much hardcoded
      // generalize this function. Rename func: Remove and...File from function definition
      .then(function([folderContentsResponse, readmeResponse]) {
        const {
          imageFileNames,
          markdownFiles
        } = extractImageAndMarkDownFileNames(folderContentsResponse);

        const sortedMarkdownFiles = sortMarkDownFilesByTableOfContents(
          markdownFiles
        );
        debugger;
      })
      .catch(function(error) {
        console.log(error);
      });
  }

  function buildFileRequestUrl(encodedBookName, filename) {
    return RAW_REPO_BASE_URL + encodedBookName + "/" + filename;
  }

  function extractImageAndMarkDownFileNames(folderContentsResponse) {
    const imageFileNames = [];
    const markdownFiles = [];
    const data = folderContentsResponse.data;

    for (var i = 0; i < data.length; i++) {
      if (RegExp(/.+\.(png|jpg)/).test(data[i].name)) {
        imageFileNames.push(data[i].name);
      }
      if (RegExp(/.+\.(md)/).test(data[i].name)) {
        markdownFiles.push(data[i].name);
      }
    }

    return {
      imageFileNames,
      markdownFiles
    };
  }

  function sortMarkDownFilesByTableOfContents(markdownFiles) {
    var tocFileNameOrder = ["readme", "toc", "foreword", "ch", "ap"];
    const mdFilesSortedByToc = [];

    tocFileNameOrder.forEach(searchFileName => {
      mdFilesSortedByToc.push(
        ...filterFilesByFileName(markdownFiles, searchFileName)
      );
    });

    return mdFilesSortedByToc;
  }

  function filterFilesByFileName(markdownFiles, searchFileName) {
    const temp = markdownFiles.filter(
      fileName => fileName.indexOf(searchFileName) !== -1
    );
    return temp.sort(); // sort numbered files (nameINDEX.md)
  }
</script>

<style>

</style>

<div>
  <select
    on:change={bookSelectionChanged}
    id="selectbook"
    class="m-2 border-solid border-blue-200 border-2">
    {#each books as book}
      <option>{book.name}</option>
    {/each}
  </select>
  <br />
   {selectedBookName}
</div>
