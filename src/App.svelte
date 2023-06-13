<script>
  import { onMount } from "svelte";
  import "bootstrap/dist/css/bootstrap.min.css";
  import DevExpress from "devextreme";

  let jsonData = [];
  let gridData = [];

  onMount(async () => {
    const response = await fetch(
      "https://api.recruitly.io/api/candidate?apiKey=TEST27306FA00E70A0F94569923CD689CA9BE6CA"
    );
    const responseData = await response.json();
    jsonData = responseData.data;

    gridData = jsonData.map((item) => ({
      id: item.id,
      firstName: item.firstName,
      surname: item.surname,
      email: item.email,
      mobile: item.mobile,
      fileId: item.fileId // Assuming you have a fileId property in the data
    }));

    const columns = [
      { dataField: "id", caption: "ID", width: 250 },
      { dataField: "firstName", caption: "Full Name", width: 200 },
      { dataField: "surname", caption: "Surname", width: 200 },
      { dataField: "email", caption: "Email", width: 200 },
      { dataField: "mobile", caption: "Mobile", width: 150 },
      // Add the file button column
      {
        caption: "Actions",
        width: 150,
        cellTemplate: function (container, options) {
          const downloadButton = document.createElement("a");
          downloadButton.className = "btn btn-success btn-sm";
          downloadButton.textContent = "Download";
          downloadButton.addEventListener("click", () => {
            downloadCV(options.data.fileId); // Use fileId instead of cvid
          });
          container.appendChild(downloadButton);
        },
      },
      // Define other columns as needed
    ];

    const dataGrid = new DevExpress.ui.dxDataGrid(document.getElementById("dataGrid"), {
      dataSource: gridData,
      columns: columns,
      showBorders: true,
      filterRow: {
        visible: true,
      },
      paging: {
        pageSize: 10,
      },
      onInitialized: () => {},
    });
  });

  async function downloadCV(fileId) {
    console.log("Data being sent to API:", fileId);
    try {
      const response = await fetch(
        `https://api.recruitly.io/api/cloudfile/download?cloudFileId=${fileId}&apiKey=TEST27306FA00E70A0F94569923CD689CA9BE6CA`
      );

      if (response.ok) {
        const blob = await response.blob();
        const url = URL.createObjectURL(blob);
        const link = document.createElement("a");
        link.href = url;
        link.download = "name.pdf"; // Replace with the desired file name
        link.click();
        URL.revokeObjectURL(url);
      } else {
        console.error("Failed to download CV.");
      }
    } catch (error) {
      console.error("Error while downloading CV:", error);
    }
  }
</script>

<style>
  #dataGrid {
    height: 400px;
  }
</style>

<h1 style="color:blue;">Job Candidate Details</h1>

<div id="dataGrid"></div>
