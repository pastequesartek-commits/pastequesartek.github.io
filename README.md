<input type="file" id="imageInput">
<script>
async function infer(imageFile) {
  const formData = new FormData();
  formData.append("file", imageFile);

  const res = await fetch("https://detect.roboflow.com/PROJET/3?api_key=6mVMfzMgUEMfEVrcl0HN", {
    method: "POST",
    body: formData
  });

  const result = await res.json();
  console.log(result);
}
</script>
