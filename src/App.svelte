<script>
  import { onMount, afterUpdate, onDestroy } from "svelte";
  import "bootstrap/dist/css/bootstrap.min.css";
  import DevExpress from "devextreme";

  let jsonData = [];
  let gridData = [];
  let isCVUploadPopupVisible = false;
  let isViewCvPopupVisible = false;
  let cvHtml = "";

  onMount(async () => {
    const response = await fetch(
      "https://api.recruitly.io/api/candidate?apiKey=TEST9349C0221517DA4942E39B5DF18C68CDA154"
    );
    const responseData = await response.json();
    jsonData = responseData.data;

    gridData = jsonData.map((item) => ({
      id: item.id,
      firstName: item.firstName,
      surname: item.surname,
      email: item.email,
      mobile: item.mobile,
    }));

    const columns = [
      { dataField: "id", caption: "ID", width: 250 },
      { dataField: "firstName", caption: "Full Name", width: 200 },
      { dataField: "surname", caption: "Surname", width: 200 },
      { dataField: "email", caption: "Email", width: 200 },
      { dataField: "mobile", caption: "Mobile", width: 150 },
      {
        caption: "Actions",
        width: 400,
        cellTemplate: function (container, options) {
          const uploadButton = document.createElement("button");
          uploadButton.innerText = "Upload CV";
          uploadButton.addEventListener("click", () => {
            isCVUploadPopupVisible = true;
          });
          container.appendChild(uploadButton);

          const downloadButton = document.createElement("button");
          downloadButton.innerText = "Download CV";
          downloadButton.addEventListener("click", async () => {
            const cvResponse = await fetch(
              `https://api.recruitly.io/api/candidatecv/${options.data.id}?apiKey=TEST27306FA00E70A0F94569923CD689CA9BE6CA`
            );
            if (cvResponse.ok) {
              const cvData = await cvResponse.json();
              const cvId = cvData.internal.cloudFile.id;
              const downloadLink = `https://api.recruitly.io/api/cloudfile/download?cloudFileId=${cvId}&apiKey=TEST45684CB2A93F41FC40869DC739BD4D126D77`;
              window.open(downloadLink);
            } else {
              alert("Failed to fetch CV file.");
            }
          });
          container.appendChild(downloadButton);

          const viewButton = document.createElement("button");
          viewButton.innerText = "View CV";
          viewButton.addEventListener("click", async () => {
            const cvResponse = await fetch(
              `https://api.recruitly.io/api/candidatecv/${options.data.id}?apiKey=TEST27306FA00E70A0F94569923CD689CA9BE6CA`
            );
            if (cvResponse.ok) {
              const cvData = await cvResponse.json();
              cvHtml = cvData.html;
              if (cvHtml) {
                isViewCvPopupVisible = true;
              } else {
                alert("CV file not found.");
              }
            } else {
              alert("Failed to fetch CV file.");
            }
          });
          container.appendChild(viewButton);
        },
        width: 400,
      },
      // Add other columns as needed
    ];

    const dataGrid = new DevExpress.ui.dxDataGrid(
      document.getElementById("dataGrid"),
      {
        dataSource: gridData,
        columns: columns,
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
              e.data.firstName = responseData.fistName;
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
              firstName: e.newData.firstName === undefined ? e.oldData.firstName : e.newData.firstName,
              surname: e.newData.surname === undefined ? e.oldData.surname : e.newData.surname,
              email: e.newData.email === undefined ? e.oldData.email : e.newData.email,
              mobile: e.newData.mobile === undefined ? e.oldData.mobile : e.newData.mobile,
            };

            console.log(newData);
            const response = await fetch(
              `https://api.recruitly.io/api/candidate?apiKey=TEST9349C0221517DA4942E39B5DF18C68CDA154`,
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
              const updatedItemIndex = gridData.findIndex((item) => item.id === e.key);
              gridData.push(e.newData);
              gridData[updatedItemIndex] = e.newData;
              dataGrid.refresh();
            } else {
              console.error("Failed to update record:", responseData.error);
            }
          } catch (error) {
            console.error("Failed to update record:", error);
          }
        },
        onRowRemoving: async (e) => {
          console.log("Data being sent to API:", e.data);
          try {
            const response = await fetch(
              `https://api.recruitly.io/api/candidate/${e.data.id}?apiKey=TEST27306FA00E70A0F94569923CD689CA9BE6CA`,
              {
                method: "DELETE",
              }
            );
            if (response.ok) {
              const removedItemIndex = gridData.findIndex((item) => item.id === e.key);
              if (removedItemIndex > -1) {
                gridData.splice(removedItemIndex, 1);
                dataGrid.refresh();
              }
            } else {
              console.error("Failed to delete record.");
            }
          } catch (error) {
            console.error("Failed to delete record:", error);
          }
        },
        onInitialized: () => {},
      }
    );

    let isCVUploadPopupVisible = false;
    let isViewCvPopupVisible = false;

    function openUploadPopup(id) {
      isCVUploadPopupVisible = true;
      selectedCandidateId = id;
    }

    function closeUploadPopup() {
      isCVUploadPopupVisible = false;
    }

    function openViewCvPopup(id) {
      isViewCvPopupVisible = true;
      selectedCandidateId = id;
    }

    function closeViewCvPopup() {
      isViewCvPopupVisible = false;
    }
</script>

<style>
  body {
    font-family: Arial, sans-serif;
    background-color: #f8f9fa;
  }

  .btn {
    margin-top: 10px;
  }

  #dataGrid {
    margin-top: 20px;
    background-color: #ffffff;
    border: 1px solid #dee2e6;
    border-radius: 4px;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
  }

  .cv-popup {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 9999;
    background-color: rgba(0, 0, 0, 0.5);
  }

  .cv-popup-content {
    max-width: 80%;
    max-height: 80%;
    background-color: #fff;
    padding: 20px;
    overflow: auto;
    border-radius: 4px;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
  }

  .cv-popup-close {
    position: absolute;
    top: 10px;
    right: 10px;
    cursor: pointer;
  }
</style>

<div id="dataGrid"></div>

{#if isCVUploadPopupVisible}
  <div class="cv-popup">
    <div class="cv-popup-content">
      <h3>Upload CV</h3>
      <input type="file" accept=".pdf,.doc,.docx" />
      <button on:click={closeUploadPopup}>Close</button>
    </div>
  </div>
{/if}

{#if isViewCvPopupVisible}
  <div class="cv-popup">
    <div class="cv-popup-content">
      <h3>View CV</h3>
      <div>
        <!-- Display CV content here -->
      </div>
      <button on:click={closeViewCvPopup}>Close</button>
    </div>
  </div>
{/if}
