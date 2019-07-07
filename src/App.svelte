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

  let selectedBookName = "";
  let selectedBookIndex = 0;
  let selectedChapterFileName = "README.md";

  // request on application start
  selectFirstBook();
  requestBookFolderContentsOfSelectedBook();

  // insert dummy data
  // for (let i = 0; i < books.length; i++) {
  //   const book = books[i];
  //   book.mdFileNames.push(Math.random());
  //   book.mdFileNames.push(Math.random());
  //   book.mdFileNames.push(Math.random());
  // }

  function onChapterSelectionChanged(selectEvent) {
    selectChapter(selectEvent);
    requestMarkdownFile();
  }

  function selectChapter(selectEvent) {
    const selectedChapterIndex = selectEvent.target.selectedIndex;
    selectChapterByIndex(selectedChapterIndex);
  }

  function selectFirstChapter() {
    selectChapterByIndex(0);
  }

  function selectChapterByIndex(selectedChapterIndex) {
    selectedChapterFileName =
      books[selectedBookIndex].mdFileNames[selectedChapterIndex];
  }

  function requestMarkdownFile() {
    if (!selectedChapterFileName) {
      return;
    }

    const encodedBookName = encodeURIComponent(selectedBookName);
    const fileRequestUrl = buildFileUrl(
      encodedBookName,
      selectedChapterFileName
    );
    axios
      .get(fileRequestUrl)
      .then(chapterFileResponse => {
        let html = marked(chapterFileResponse.data);
        html = injectImageUrlsIntoHtml(html, encodedBookName);
        const contentElement = document.getElementById("content");
        contentElement.innerHTML = html;
      })
      .catch(function(error) {
        console.log(error);
      });
  }

  function buildFileUrl(encodedBookName, fileName) {
    return RAW_REPO_BASE_URL + encodedBookName + "/" + fileName;
  }

  function injectImageUrlsIntoHtml(html, encodedBookName) {
    const imageFileNames = books[selectedBookIndex].imageFileNames;

    for (var j = 0; j < imageFileNames.length; j++) {
      var imageUrl = buildFileUrl(encodedBookName, imageFileNames[j]);
      html = html.replace(imageFileNames[j], imageUrl);
    }
    return html;
  }

  function onBookSelectionChanged(selectEvent) {
    selectBook(selectEvent);
    requestBookFolderContentsOfSelectedBook();
  }

  function selectBook(selectEvent) {
    selectedBookIndex = selectEvent.target.selectedIndex;
    selectBookByIndex(selectedBookIndex);
  }

  function selectFirstBook() {
    selectedBookIndex = 0;
    selectBookByIndex(0);
  }

  function selectBookByIndex(selectedBookIndex) {
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

        selectFirstChapter();
        requestMarkdownFile();
      })
      .catch(function(error) {
        console.log(error);
      });
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

  function filterFilesByFileName(markdownFiles, searchFileName) {
    const temp = markdownFiles.filter(
      fileName => fileName.toLowerCase().indexOf(searchFileName) !== -1
    );
    return temp.sort(); // sort numbered files (nameINDEX.md)
  }

  function cacheImageAndSortedmdFileNames(imageFileNames, sortedMdFileNames) {
    books[selectedBookIndex].imageFileNames = imageFileNames;
    books[selectedBookIndex].mdFileNames = sortedMdFileNames;
  }
</script>

<style>

</style>

<div>
  <select
    on:change={onBookSelectionChanged}
    class="m-2 border-solid border-blue-200 border-2">
    {#each books as book}
      <option>{book.name}</option>
    {/each}
  </select>
  <br />

  {#if books[selectedBookIndex].mdFileNames.length > 0}
    <select
      on:change={onChapterSelectionChanged}
      class="m-2 border-solid border-blue-200 border-2">
      {#each books[selectedBookIndex].mdFileNames as mdFileName}
        <option>{mdFileName}</option>
      {/each}
    </select>
  {/if}

  <main class="bg-white m-4 p-4 rounded-lg">
    <div id="content" class="markdown-body">
      Helping marketers serve unmatched cross-phase personalized experiences at
      every step of the FirstSpriti Digital Experience Platform powers
      enterprise-class. Clicking on this link which refers to B2B Marketing
      awards shortlist will take you to the awards page of the FirstSpriti
      Digital Experience Platform. Spriti introduced new capabilities of the
      FirstSpriti Digital Experience Platform. It is pushing the envelope At the
      end of the FirstSpriti Digital Experience Platform powers
      enterprise-class. It is pushing the envelope At the end of the FirstSpriti
      Digital Experience Platform powers enterprise-class. Helping marketers
      serve unmatched cross-phase personalized experiences at every step of the
      FirstSpriti Digital Experience Platform powers enterprise-class. Clicking
      on this link which refers to B2B Marketing awards shortlist will take you
      to the awards page of the customer journey. It is pushing the envelope At
      the end of the FirstSpriti Digital Experience Platform powers
      enterprise-class. Clicking on this link which refers to B2B Marketing
      awards shortlist will take you to the awards page of the customer journey.
      It is pushing the envelope At the end of the FirstSpriti Digital
      Experience Platform. These innovations help CMOs challenged with the
      delivery of omnichannel digital experiences for some of the FirstSpriti
      Digital Experience Platform powers enterprise-class. Spriti introduced new
      capabilities of the FirstSpriti Digital Experience Platform powers
      enterprise-class.
    </div>
  </main>
</div>
