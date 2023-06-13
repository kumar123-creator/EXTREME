<script>
  import { onMount, createEventDispatcher } from "svelte";
  import "bootstrap/dist/css/bootstrap.min.css";
  import DevExpress from "devextreme";

  let jsonData = [];
  let gridData = [];
  let isCVUploadPopupVisible = false;
  let isViewCvPopupVisible = false;
  let selectedRowData = null;
  let selectedCVId = null;

  async function uploadCV(file) {
    // Perform further actions with the uploaded file

    // Example: Update the backend API URL with the file upload
    const formData = new FormData();
    formData.append("file", file);

    if (selectedRowData) {
      const uploadCandidateId = selectedRowData.id; // Get the candidate ID from selectedRowData

      try {
        const response = await fetch(
          `https://api.recruitly.io/api/cloudfile/download?cloudFileId=${selectedCVId}&apiKey=TEST45684CB2A93F41FC40869DC739BD4D126D77`,
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

  async function downloadCV(CVId) {
    console.log("cvid:", CVId);
    try {
      const response = await fetch(
        `https://api.recruitly.io/api/cloudfile/download?cloudFileId=${CVId}&apiKey=TEST27306FA00E70A0F94569923CD689CA9BE6CA`
      );

      if (response.ok) {
        const blob = await response.blob();
        const url = URL.createObjectURL(blob);
        const link = document.createElement("a");
        link.href = url;
        link.download = "CV.pdf"; // Replace with the desired file name
        link.click();
        URL.revokeObjectURL(url);
      } else {
        console.error("Failed to download CV.");
      }
    } catch (error) {
      console.error("Error while downloading CV:", error);
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
      id: item.id,
      firstName: item.firstName,
      surname: item.surname,
      email: item.email,
      mobile: item.mobile,
      cvId: item.cvId, // assuming cvId is the property containing the CV file ID
    }));

    const dataGrid = new DevExpress.ui.dxDataGrid(
      document.getElementById("dataGrid"),
      {
        dataSource: gridData,
        columns: [
          { dataField: "id", caption: "ID", width: 250 },
          { dataField: "firstName", caption: "First Name", width: 180 },
          { dataField: "surname", caption: "Surname", width: 180 },
          { dataField: "email", caption: "Email", width: 180 },
          { dataField: "mobile", caption: "Mobile", width: 100 },
          {
            caption: "Actions",
            width: 350,
            cellTemplate: function (container, options) {
              const cvUploadButton = document.createElement("button");
              cvUploadButton.innerText = "CV Upload";
              cvUploadButton.classList.add("btn", "btn-primary", "mr-2");
              cvUploadButton.addEventListener("click", function () {
                const rowData = options.data;
                selectedRowData = rowData;
                selectedCVId = rowData.cvId; // Assuming cvId is the property containing the CV file ID
                isCVUploadPopupVisible = true;
              });

              const cvDownloadButton = document.createElement("button");
              cvDownloadButton.innerText = "CV Download";
              cvDownloadButton.classList.add("btn", "btn-info", "mr-2");
              cvDownloadButton.addEventListener("click", function () {
                const rowData = options.data;
                const cvId = rowData.cvId; // Assuming cvId is the property containing the CV file ID
                downloadCV(cvId);
              });

              const viewCVButton = document.createElement("button");
              viewCVButton.innerText = "View CV";
              viewCVButton.classList.add("btn", "btn-secondary");
              viewCVButton.addEventListener("click", function () {
                const rowData = options.data;
                selectedCVId = rowData.cvId; // Assuming cvId is the property containing the CV file ID
                isViewCvPopupVisible = true;
              });

              container.appendChild(cvUploadButton);
              container.appendChild(cvDownloadButton);
              container.appendChild(viewCVButton);
            },
          },
        ],
        showBorders: true,
        filterRow: {
          visible: true,
        },
        editing: {
          allowDeleting: true,
          allowAdding: true,
          allowUpdating: true,
          mode: "popup",
          form: {
            labelLocation: "top",
          },
          popup: {
            showTitle: true,
            title: "Row in the editing state",
          },
          texts: {
            saveRowChanges: "Save",
            cancelRowChanges: "Cancel",
            deleteRow: "Delete",
            confirmDeleteMessage:
              "Are you sure you want to delete this record?",
          },
          onSaveRowChanges: handleSave, // Bind handleSave function to the saveRowChanges event
        },
        paging: {
          pageSize: 10,
        },
        onRowInserting: async (e) => {
          // ...
        },
        onRowUpdating: async (e) => {
          // ...
        },
        onRowRemoving: async (e) => {
          // ...
        },
        onInitialized: () => {
          // Function called when the grid is initialized
          // ...
        },
      }
    );
  });
</script>

<div id="dataGrid"></div>

{#if isCVUploadPopupVisible}
<div class="popup-overlay">
  <div class="popup-content">
    <h3>Upload CV</h3>
    <input
      type="file"
      on:change="{(event) => uploadCV(event.target.files[0])}"
    />
    <button class="btn btn-primary" on:click="{() => handleClose()}">
      Close
    </button>
  </div>
</div>
{/if}
{#if isViewCvPopupVisible}
<div class="popup-overlay">
  <div class="popup-content">
    <h3>View CV</h3>
    {#if isViewCvPopupVisible}
    <div class="popup-overlay">
      <div class="popup-content">
        <h3>View CV</h3>
        <iframe src="https://api.recruitly.io/api/cloudfile/download?cloudFileId={selectedCVId}&apiKey=TEST27306FA00E70A0F94569923CD689CA9BE6CA" width="100%" height="600px"></iframe>
        <button class="btn btn-primary" on:click="{() => handleClose()}">
          Close
        </button>
      </div>
    </div>
    {/if}

    <button class="btn btn-primary" on:click="{() => handleClose()}">
      Close
    </button>
  </div>
</div>
{/if}

<style>
  .popup-overlay {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background-color: rgba(0, 0, 0, 0.5);
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .popup-content {
    background-color: white;
    padding: 20px;
    border-radius: 4px;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
    width: 400px;
  }

  .popup-content h3 {
    margin-top: 0;
  }

  .btn {
    margin-top: 10px;
  }

  #dataGrid {
    margin-top: 20px;
  }
</style>
