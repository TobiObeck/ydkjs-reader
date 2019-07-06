<script>
  export let name;

  import axios from "axios";

  var RAW_REPO_BASE_URL =
    "https://raw.githubusercontent.com/getify/You-Dont-Know-JS/master/";
  var REPO_FILES_URL =
    "https://api.github.com/repos/getify/You-Dont-Know-JS/contents/";

  const books = [
    {
      name: "up & going",
      mdFileNames: [],
      imageFileNames: [],
      mdFileNames: [],
      imageFileNames: []
    },
    {
      name: "scope & closures",
      mdFileNames: [],
      imageFileNames: []
    },
    {
      name: "this & object prototypes",
      mdFileNames: [],
      imageFileNames: []
    },
    {
      name: "types & grammar",
      mdFileNames: [],
      imageFileNames: []
    },
    {
      name: "async & performance",
      mdFileNames: [],
      imageFileNames: []
    },
    {
      name: "es6 & beyond",
      mdFileNames: [],
      imageFileNames: []
    }
  ];

  let selectedBookName = ""; // books[0];
  let selectedBookIndex = 0;
  let selectedChapterFileName = "README.md";

  function onChapterSelectionChanged(selectEvent) {
    selectChapter(selectEvent);
    requestMarkdownFile();
  }

  function selectChapter(selectEvent) {
    const selectedChapterIndex = selectEvent.target.selectedIndex;
    selectedChapterFileName =
      books[selectedBookIndex].mdFileNames[selectedChapterIndex];
  }

  function requestMarkdownFile() {
    if (!selectedChapterFileName) {
      return;
    }

    const encodedBookName = encodeURIComponent(selectedBookName);
    const fileRequestUrl = buildFileRequestUrl(
      encodedBookName,
      selectedChapterFileName
    );
    axios
      .get(fileRequestUrl)
      .then(chapterFileResponse => {
        const html = marked(chapterFileResponse.data);
        const contentElement = document.getElementById("content");
        contentElement.innerHTML = html;
      })
      .catch(function(error) {
        console.log(error);
      });
  }

  function onBookSelectionChanged(selectEvent) {
    selectBook(selectEvent);
    requestBookFolderContentsOfSelectedBook();
  }

  function selectBook(selectEvent) {
    selectedBookIndex = selectEvent.target.selectedIndex;
    selectedBookName = books[selectedBookIndex].name;
  }

  function requestBookFolderContentsOfSelectedBook() {
    if (!selectedBookName) {
      return;
    }

    const encodedBookName = encodeURIComponent(selectedBookName);
    const folderContentsQuery = REPO_FILES_URL + encodedBookName;

    axios
      .get(folderContentsQuery)
      .then(folderContentsResponse => {
        const { imageFileNames, markdownFiles } = extractImageAndmdFileNames(
          folderContentsResponse
        );

        const sortedMarkdownFiles = sortMarkDownFilesByTableOfContents(
          markdownFiles
        );

        cacheImageAndSortedmdFileNames(imageFileNames, sortedMarkdownFiles);
      })
      .catch(function(error) {
        console.log(error);
      });
  }

  function buildFileRequestUrl(encodedBookName, chapterFilename) {
    return RAW_REPO_BASE_URL + encodedBookName + "/" + chapterFilename;
  }

  function extractImageAndmdFileNames(folderContentsResponse) {
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

  function cacheImageAndSortedmdFileNames(imageFileNames, sortedMdFileNames) {
    books[selectedBookIndex].imageFileNames = imageFileNames;
    books[selectedBookIndex].mdFileNames = sortedMdFileNames;
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
  <p class="text-xl">-> {selectedBookName}</p>

  <select
    on:change={onBookSelectionChanged}
    class="m-2 border-solid border-blue-200 border-2">
    {#each books as book}
      <option>{book.name}</option>
    {/each}
  </select>
  <br />

  {#if books[selectedBookIndex].mdFileNames.length}
    <select
      on:change={onChapterSelectionChanged}
      class="m-2 border-solid border-blue-200 border-2">
      {#each books[selectedBookIndex].mdFileNames as mdFileName}
        <option>{mdFileName}</option>
      {/each}
    </select>
  {/if}

  <main id="content" class="markdown-body">asd</main>
</div>
