<script>
	import { onMount, createEventDispatcher } from "svelte";
	import "bootstrap/dist/css/bootstrap.min.css";
	import DevExpress from "devextreme";
	
	let jsonData = [];
	let gridData = [];
	let isCVUploadPopupVisible = false
	let selectedRowData = null;
	
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
			{ dataField: "surname", caption: "Surname", width: 180 },
			{ dataField: "email", caption: "Email", width: 200 },
			{ dataField: "mobile", caption: "Mobile", width: 150 },
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
				  // Implement view CV logic here
				  console.log("View CV clicked for row:", rowData);
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
		accept=".pdf,.doc,.docx"
	  />
	  <button on:click={handleSave} class="btn btn-primary">Save</button>
	  <button on:click={handleClose} class="btn btn-secondary">Close</button>
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
  }
  </style>
  