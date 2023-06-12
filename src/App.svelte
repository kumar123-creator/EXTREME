<script>
  import { onMount, createEventDispatcher } from "svelte";
  import "bootstrap/dist/css/bootstrap.min.css";
  import DevExpress from "devextreme";

  let jsonData = [];
  let gridData = [];
  let isCVUploadPopupVisible = false;
  let selectedRowData = null;
  let isViewCVPopupVisible = false;
  let cvImageUrl = "";

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

  function handleViewCVPopupClose() {
    isViewCVPopupVisible = false;
    cvImageUrl = "";
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
      cvUrl: item.cvUrl, // assuming cvUrl is the property containing the CV file URL
    }));

    const dataGrid = new DevExpress.ui.dxDataGrid(
      document.getElementById("dataGrid"),
      {
        dataSource: gridData,
        columns: [
          { dataField: "id", caption: "ID", width: 250 },
          { dataField: "firstName", caption: "First Name", width: 180 },
          { dataField: "surname", caption: "Surname", width: 1800 },
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
                isCVUploadPopupVisible = true;
              });

              const cvDownloadButton = document.createElement("button");
              cvDownloadButton.innerText = "CV Download";
              cvDownloadButton.classList.add("btn", "btn-info", "mr-2");
              cvDownloadButton.addEventListener("click", function () {
                const rowData = options.data;
                const cvUrl = rowData.cvUrl; // Assuming cvUrl is the property containing the CV file URL
                downloadCV(cvUrl);
              });

              const viewCVButton = document.createElement("button");
              viewCVButton.innerText = "View CV";
              viewCVButton.classList.add("btn", "btn-secondary");
              viewCVButton.addEventListener("click", function () {
                const rowData = options.data;
                const cvUrl = rowData.cvUrl; // Assuming cvUrl is the property containing the CV file URL
                cvImageUrl = cvUrl;
                isViewCVPopupVisible = true;
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
            confirmDeleteMessage: "Are you sure you want to delete this record?",
          },
          onSaveRowChanges: handleSave, // Bind handleSave function to the saveRowChanges event
        },
        paging: {
          pageSize: 10,
        },
        onRowInserting: async (e) => {
          console.log("Data being sent to API:", e.data);
          try {
            const response = await fetch(
              "https://api.recruitly.io/api/candidate?apiKey=TEST27306FA00E70A0F94569923CD689CA9BE6CA",
              {
                method: "POST",
                headers: {
                  "Content-Type": "application/json",
                },
                body: JSON.stringify(e.data),
              }
            );

            const responseData = await response.json();
            if (response.ok) {
              e.data.firstName = responseData.firstName;
              gridData.push(e.data);
              dataGrid.refresh();
            } else {
              console.error("Failed to add record:", responseData.error);
            }
          } catch (error) {
            console.error("Failed to add record:", error);
          }
        },
        onRowUpdating: async (e) => {
          try {
            console.log(e);
            var newData = {
              id: e.key.id,
              firstName:
                e.newData.firstName === undefined
                  ? e.oldData.firstName
                  : e.newData.firstName,
              surname:
                e.newData.surname === undefined ? e.oldData.surname : e.newData.surname,
              email: e.newData.email === undefined ? e.oldData.email : e.newData.email,
              mobile:
                e.newData.mobile === undefined ? e.oldData.mobile : e.newData.mobile,
            };

            console.log(newData);
            const response = await fetch(
              `https://api.recruitly.io/api/candidate?apiKey=TEST27306FA00E70A0F94569923CD689CA9BE6CA`,
              {
                method: "POST",
                headers: {
                  "Content-Type": "application/json",
                },
                body: JSON.stringify(newData),
              }
            );
            const responseData = await response.json();
            if (response.ok) {
              e.newData = responseData;
            } else {
              console.error("Failed to update record:", responseData.error);
            }
          } catch (error) {
            console.error("Failed to update record:", error);
          }
        },
        onRowRemoving: async (e) => {
          try {
            const response = await fetch(
              `https://api.recruitly.io/api/candidate?apiKey=TEST27306FA00E70A0F94569923CD689CA9BE6CA&id=${e.data.id}`,
              {
                method: "DELETE",
              }
            );
            if (response.ok) {
              const index = gridData.findIndex((item) => item.id === e.data.id);
              gridData.splice(index, 1);
              dataGrid.refresh();
            } else {
              console.error("Failed to delete record:", response.error);
            }
          } catch (error) {
            console.error("Failed to delete record:", error);
          }
        },
      }
    );

    dataGrid.render();
  });
</script>

<style>
  #dataGrid {
    height: 400px;
  }

  .popup-overlay {
    display: flex;
    justify-content: center;
    align-items: center;
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.5);
    z-index: 9999;
  }

  .popup-content {
    background-color: #fff;
    padding: 20px;
    border-radius: 4px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
  }
</style>

<h1>Recruitly</h1>

<div id="dataGrid"></div>

{#if isCVUploadPopupVisible}
<div class="popup-overlay">
  <div class="popup-content">
    <h3>CV Upload</h3>
    <input type="file" accept=".pdf,.doc,.docx,.png,.jpg,.jpeg" on:change="{(e) => uploadCV(e.target.files[0])}">
    <button class="btn btn-primary" on:click="{handleClose}">Close</button>
  </div>
</div>
{/if}

{#if isViewCVPopupVisible}
<div class="popup-overlay">
  <div class="popup-content">
    <h3>View CV</h3>
    <img src="{cvImageUrl}" alt="CV Image" on:error="{handleViewCVPopupClose}" />
    <button id="viewCVPopupCloseButton" on:click="{handleViewCVPopupClose}" class="btn btn-secondary">Close</button>
  </div>
</div>
{/if}
