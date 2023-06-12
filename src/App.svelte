<script>
  import { onMount, createEventDispatcher } from "svelte";
  import "bootstrap/dist/css/bootstrap.min.css";
  import DevExpress from "devextreme";

  let jsonData = [];
  let gridData = [];
  let isCVUploadPopupVisible = false;
  let isViewCvPopupVisible = false;
  let selectedRowData = null;
  let viewCvUrl = null;

  async function uploadCV(file) {
    // Perform further actions with the uploaded file
    // Example: Update the backend API URL with the file upload
    const formData = new FormData();
    formData.append("file", file);

    if (selectedRowData) {
      const uploadCandidateId = selectedRowData.id; // Get the candidate ID from selectedRowData

      try {
        const response = await fetch(
          `https://api.recruitly.io/api/candidatecv/upload?apiKey=TEST27306FA00E70A0F94569923CD689CA9BE6CA&candidateId=${uploadCandidateId}`,
          {
            method: "POST",
            body: formData,
          }
        );

        if (response.ok) {
          console.log("CV uploaded successfully");
          // Perform any additional actions upon successful upload
        } else {
          console.error("CV upload failed.");
          // Handle the error accordingly
        }
      } catch (error) {
        console.error("CV upload error:", error);
        // Handle the error accordingly
      }
    }

    // Close the CV upload popup
    isCVUploadPopupVisible = false;
  }

  async function downloadCV(cvUrl) {
    try {
      const response = await fetch(
        `https://api.recruitly.io/api/cloudfile/download?cloudFileId=${cvUrl}&apiKey=apiKey=TEST27306FA00E70A0F94569923CD689CA9BE6CA`
      );

      if (response.ok) {
        // Extract the file name from the response headers
        const contentDisposition = response.headers.get("content-disposition");
        const fileName = contentDisposition
          ? contentDisposition.split("filename=")[1]
          : "CV_File";

        const blob = await response.blob();
        const url = URL.createObjectURL(blob);
        const link = document.createElement("a");
        link.href = url;
        link.download = fileName;
        link.click();
        alert("CV downloaded successfully!");
      } else {
        console.error("CV download failed.");
        // Handle the error accordingly
      }
    } catch (error) {
      console.error("CV download error:", error);
      // Handle the error accordingly
    }
  }

  function handleSave() {
    // Perform save logic
    // In this case, we're updating the backend API URL in the handleSave function
    console.log("Save clicked");

    // Close the CV upload popup
    isCVUploadPopupVisible = false;
  }

  function handleClose() {
    // Perform close logic
    console.log("Close clicked");

    // Close the CV upload popup
    isCVUploadPopupVisible = false;
  }

  function handleViewCvClose() {
    // Close the CV view popup
    isViewCvPopupVisible = false;
  }

  const dispatch = createEventDispatcher();

  onMount(async () => {
    const response = await fetch(
      "https://api.recruitly.io/api/candidate?apiKey=TEST27306FA00E70A0F94569923CD689CA9BE6CA"
    );
    const responseData = await response.json();
    jsonData = responseData.data;

    gridData = jsonData.map((item) => ({
      ...item,
      actions: {
        viewCVButton: {
          text: "View CV",
          onClick: function (e) {
            const rowData = e.data;
            selectedRowData = rowData;
            isViewCvPopupVisible = true;
            viewCvUrl = rowData.cvUrl;
          },
        },
        uploadCVButton: {
          text: "Upload CV",
          onClick: function (e) {
            const rowData = e.data;
            selectedRowData = rowData;
            isCVUploadPopupVisible = true;
          },
        },
      },
    }));

    DevExpress.ui.notify("Data loaded successfully", "success", 1000);
  });
</script>

<main>
  <h1>Candidate Grid</h1>
  <DevExpress.DataGrid
    dataSource={gridData}
    allowColumnReordering={true}
    showBorders={true}
  >
    <DevExpress.Column dataField="id" caption="ID"></DevExpress.Column>
    <DevExpress.Column dataField="name" caption="Name"></DevExpress.Column>
    <DevExpress.Column
      dataField="actions.viewCVButton"
      caption="View CV"
      cellTemplate={(container, options) => {
        const viewCVButton = document.createElement("button");
        viewCVButton.innerText = "View CV";
        viewCVButton.classList.add("btn", "btn-secondary");
        viewCVButton.addEventListener("click", function () {
          const rowData = options.data;
          selectedRowData = rowData;
          isViewCvPopupVisible = true;
          viewCvUrl = rowData.cvUrl;
        });

        container.appendChild(viewCVButton);
      }}
    ></DevExpress.Column>
    <DevExpress.Column
      dataField="actions.uploadCVButton"
      caption="Upload CV"
      cellTemplate={(container, options) => {
        const uploadCVButton = document.createElement("button");
        uploadCVButton.innerText = "Upload CV";
        uploadCVButton.classList.add("btn", "btn-primary");
        uploadCVButton.addEventListener("click", function () {
          const rowData = options.data;
          selectedRowData = rowData;
          isCVUploadPopupVisible = true;
        });

        container.appendChild(uploadCVButton);
      }}
    ></DevExpress.Column>
  </DevExpress.DataGrid>

  {#if isCVUploadPopupVisible}
    <div class="popup-overlay">
      <div class="popup-content">
        <h2>Upload CV</h2>
        <input type="file" on:change={event => uploadCV(event.target.files[0])} />
        <button on:click={handleSave} class="btn btn-primary">Save</button>
        <button on:click={handleClose} class="btn btn-secondary">Close</button>
      </div>
    </div>
  {/if}

  {#if isViewCvPopupVisible}
    <div class="popup-overlay">
      <div class="popup-content">
        <h2>View CV</h2>
        <img src={viewCvUrl} alt="CV" />
        <button on:click={handleViewCvClose} class="btn btn-secondary">Close</button>
      </div>
    </div>
  {/if}
</main>

<style>
  .popup-overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
    background-color: rgba(0, 0, 0, 0.5);
  }

  .popup-content {
    background-color: white;
    padding: 20px;
    border-radius: 8px;
    display: flex;
    flex-direction: column;
    align-items: center;
  }
</style>
