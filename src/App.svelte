<script>
  import { onMount, createEventDispatcher } from "svelte";
  import "bootstrap/dist/css/bootstrap.min.css";
  import DevExpress from "devextreme";

  let jsonData = [];
  let gridData = [];
  let isCVUploadPopupVisible = false;
  let isViewCvPopupVisible = false;
  let selectedRowData = null;
  let selectedCvUrl = null;

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

  async function viewCV(candidateId) {
    const cvUrl = `https://api.recruitly.io/api/cloudfile/download?cloudFileId=b12d3423-5541-49a6-b3a1-8ed273e50f53&apiKey=TEST27306FA00E70A0F94569923CD689CA9BE6CA`;

    try {
      const response = await fetch(cvUrl);
      if (response.ok) {
        const blob = await response.blob();
        const url = URL.createObjectURL(blob);
        selectedCvUrl = url; // Set the selectedCvUrl to the object URL of the file
        isViewCvPopupVisible = true; // Show the view CV popup
      } else {
        console.error("CV fetch failed.");
        // Handle the error accordingly
      }
    } catch (error) {
      console.error("CV fetch error:", error);
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
    isViewCvPopupVisible = false;
  }
  async function downloadCV(cvid) {
  try {
    const response = await fetch(
      `https://api.recruitly.io/api/cloudfile/download?cloudFileId=${cvid}&apiKey=TEST27306FA00E70A0F94569923CD689CA9BE6CA`
    );

    if (response.ok) {
      // Extract the file name from the response headers
      const contentDisposition = response.headers.get("content-disposition");
      const fileName = contentDisposition
        ? contentDisposition.split("filename=")[1]
        : "CV_File";

      // Create a temporary download link and trigger the download
      const blob = await response.blob();
      const url = URL.createObjectURL(blob);
      const link = document.createElement("a");
      link.href = url;
      link.download = fileName;
      link.click();

      // Show success message
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
          { dataField: "firstName", caption: "First Name" },
          { dataField: "surname", caption: "Surname" },
          { dataField: "email", caption: "Email" },
          { dataField: "mobile", caption: "Mobile" },
          {
            dataField: "cvUrl",
            caption: "CV",
            cellTemplate: function (container, options) {
              const button = document.createElement("button");
              button.className = "btn btn-secondary";
              button.innerText = "View";
              button.addEventListener("click", function () {
                selectedRowData = options.data; // Store the selected row data
                const candidateId = options.data.id; // Assuming 'id' is the candidate ID property
                viewCV(candidateId);
              });
              container.appendChild(button);
            },
          },
        ],
      }
    );
  });
</script>

<style>
  .popup-overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.5);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 9999;
  }

  .popup-content {
    background-color: white;
    padding: 20px;
    max-width: 80%;
    max-height: 80%;
    overflow: auto;
  }
</style>

<h1>Candidate CVs</h1>

<div id="dataGrid"></div>

{#if isCVUploadPopupVisible}
<div class="popup-overlay">
  <div class="popup-content">
    <h3>Upload CV</h3>
    <input type="file" accept=".pdf" on:change="{(e) => uploadCV(e.target.files[0])}" />
    <button class="btn btn-primary" on:click="{() => handleSave()}">
      Save
    </button>
    <button class="btn btn-secondary" on:click="{() => handleClose()}">
      Close
    </button>
  </div>
</div>
{/if}

{#if isViewCvPopupVisible}
<div class="popup-overlay">
  <div class="popup-content">
    <h3>View CV</h3>
    <iframe src="{selectedCvUrl}" width="100%" height="600px"></iframe>
    <button class="btn btn-primary" on:click="{() => handleClose()}">
      Close
    </button>
  </div>
</div>
{/if}
