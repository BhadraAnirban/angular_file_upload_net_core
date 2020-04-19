# angular_file_upload_net_core
Upload File or Attach file using Angular and .net core working in AWS as well

The process is to send the base64 string from Angular using POST call in the FormBody and at .net core convert the string to byte.

## At .net core -

### In the controller:
```
        [HttpPost("ConvertFileInByte"), DisableRequestSizeLimit]
        public IActionResult ConvertFileInByte([FromBody] FileModel fileModel)
        {
            try
            {
                var value = fileModel.Data;                 
                return Ok(new { FileName = fileModel.Name, Data = Convert.FromBase64String(value) });
                
               /// Convert.FromBase64String(value) will convert the base64 string to byte
            }
            catch (Exception e)
            {
                throw e;
            }

        }
        
 public class FileModel
{
    public string Data { get; set; }
    public string Name { get; set; }
}
        
```
Convert.FromBase64String(value) will convert the base64 string to byte


