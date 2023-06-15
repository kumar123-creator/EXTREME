<script>
  import { onMount } from "svelte";
  import "bootstrap/dist/css/bootstrap.min.css";
  import DevExpress from "devextreme";

  let jsonData = [];
  let gridData = [];
  let isCVUploadPopupVisible = false;
  let isCVViewPopupVisible = false;
  let selectedRowData = null;
  let cvWindow = null;

  
   async function uploadCV(file) {
	  // Perform further actions with the uploaded file
  
	  // Example: Update the backend API URL with the file upload
	  const formData = new FormData();
	  formData.append("file", file);
  
	  if (selectedRowData) {
		const uploadCandidateId = selectedRowData.id; // Get the candidate ID from selectedRowData
  
		try {
		  const response = await fetch(
			`https://api.recruitly.io/api/candidatecv/upload?apiKey=TEST1236C4CF23E6921C41429A6E1D546AC9535E&candidateId=${uploadCandidateId}`,
			{
			  method: "POST",
			  body: formData,
			}
		  );
  
		  if (response.ok) {
        alert("file uploaded successfully");
			console.log("CV uploaded successfully!");
      console.log(uploadCandidateId);
			// Perform any additional actions upon successful upload
		  } else {
			console.error("CV upload fail.");
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
  
	  // Close the CV upload popup or CV view popup
	  isCVUploadPopupVisible = false;
	  isCVViewPopupVisible = false;
	}
	
	function openPopup(cvHtml) {
    if (cvWindow && !cvWindow.closed) {
      cvWindow.close(); // Close the existing popup if it's already open
    }

    cvWindow = window.open("", "_blank", "width=800,height=600"); // Open a new popup window
    cvWindow.document.write(cvHtml);
    cvWindow.document.close();
  }
	
  function getFileFromUser() {
    return new Promise((resolve, reject) => {
      const input = document.createElement("input");
      input.type = "file";
      input.accept = ".pdf,.doc,.docx";
      input.onchange = () => {
        if (input.files && input.files.length > 0) {
          resolve(input.files[0]);
        } else {
          reject(new Error("No file selected"));
        }
      };
      input.click();
    });
  }


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
        const cvUploadButton = document.createElement("button");
	cvUploadButton.innerText = "CV Upload";
	cvUploadButton.classList.add("btn", "btn-success", "mr-2");
	cvUploadButton.style.marginRight = "10px";
	cvUploadButton.addEventListener("click", async function () {
        const rowData = options.data;
	selectedRowData = rowData;
        isCVUploadPopupVisible = true;
         try {
      const file = await getFileFromUser(); // Prompt the user to select a file
      if (file) {
        await uploadCV(file); // Upload the selected file
      }
    } catch (error) {
      console.error("CV upload error:", error);
      // Handle the error accordingly
    }
  });
  container.appendChild(cvUploadButton);


          const downloadButton = document.createElement("button");
          downloadButton.classList.add("btn", "btn-success", "mr-2");
          downloadButton.innerText = "Download CV";
	  downloadButton.style.marginRight = "10px";
          downloadButton.addEventListener("click", async () => {
            const cvResponse = await fetch(
              `https://api.recruitly.io/api/candidatecv/${options.data.id}?apiKey=TEST27306FA00E70A0F94569923CD689CA9BE6CA`
            );
            if (cvResponse.ok) {
              const cvData = await cvResponse.json();
              console.log(cvData);
              const cvId = cvData.internal.cloudFile.id;
              console.log(cvData.internal.cloudFile);
              console.log(cvId);

              const downloadLink = `https://api.recruitly.io/api/cloudfile/download?cloudFileId=${cvId}&apiKey=TEST45684CB2A93F41FC40869DC739BD4D126D77`;
              window.open(downloadLink);
              alert("file downloaded successfully");
            } else {
              alert("Failed to fetch CV file.");
            }
          });
          container.appendChild(downloadButton);

          const viewButton = document.createElement("button");
          viewButton.classList.add("btn", "btn-primary", "mr-2");
          viewButton.innerText = "View CV";
	  viewButton.style.marginRight = "10px";
          viewButton.addEventListener("click", async () => {
           const cvResponse = await fetch(
                                          `https://api.recruitly.io/api/candidatecv/${options.data.id}?apiKey=TEST27306FA00E70A0F94569923CD689CA9BE6CA`
                                           );
          if (cvResponse.ok) {
         const cvData = await cvResponse.json();
         const cvHtml = cvData.html;
           if (cvHtml) {
                         openPopup(cvHtml); // Open the CV view popup
                         } else {
                  alert("CV file not found.");
                                }
                 } else {
                 alert("Failed to fetch CV.");
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
  });
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
    display: none;
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
	
	.action-buttons {
    display: flex;
    justify-content: space-between;
    margin-top: 10px;
  }

  .action-buttons button {
    margin-right: 10px;
  }

  .upload-button {
    background-color: #5cb85c;
    color: white;
  }

  .download-button {
    background-color: #5bc0de;
    color: white;
  }

  .view-button {
    background-color: #337ab7;
    color: white;
  }

</style>

<div id="dataGrid"></div>

{#if isCVUploadPopupVisible}
<div class="cv-popup">
  <div class="cv-popup-content">
  <h3>Upload CV</h3>
  <input
    type="file"
    on:change="{(event) => uploadCV(event.target.files[0])}"
    accept=".pdf,.doc,.docx"
  />
  <button on:click={handleSave} class="btn btn-primary">Save</button>
  <button on:click={handleClose} class="btn btn-secondary">Close</button>
  </div>
</div>
{/if}
