<h1 style="color:blue;">Job Candidate Details</h1> 
<script>
  import { onMount } from "svelte";
  import "bootstrap/dist/css/bootstrap.min.css";
  import DevExpress from "devextreme";
  
  let jsonData = [];
  let gridData = [];
  let isCVUploadPopupVisible = false;
	let selectedRowData = null;
  
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
  
    const dataGrid = new DevExpress.ui.dxDataGrid(document.getElementById("dataGrid"), {
      dataSource: gridData,
      columns: [
        { dataField: "id", caption: "ID", width: 250 },
        { dataField: "firstName", caption: "Full Name", width: 200 },
        { dataField: "surname", caption: "Surname", width: 200 },
        { dataField: "email", caption: "Email", width: 200 },
        { dataField: "mobile", caption: "Mobile", width: 150 },
        {
			  caption: "Files",
			  width: 400,
			  cellTemplate: function (container, options) {
				const cvUploadButton = document.createElement("button");
				cvUploadButton.innerText = "CV Upload";
				cvUploadButton.classList.add("btn", "btn-success", "mr-2");
				cvUploadButton.addEventListener("click", function () {
				  const rowData = options.data;
				  selectedRowData = rowData;
				  isCVUploadPopupVisible = true;
          container.appendChild(cvUploadButton);
				});
      }}
	
        // Define other columns as needed
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
            firstName: e.newData.firstName === undefined ? e.oldData.firstName : e.newData.firstName,
            surname: e.newData.surname === undefined ? e.oldData.surname : e.newData.surname,
            email: e.newData.email === undefined ? e.oldData.email : e.newData.email,
            mobile: e.newData.mobile === undefined ? e.oldData.mobile : e.newData.mobile,
          }
  
          console.log(newData)
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
            onInitialized: () => {
  
           },
          });
           });

                
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
			console.log("CV uploaded successfully!");
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
	
</script>
<div id="dataGrid"></div>