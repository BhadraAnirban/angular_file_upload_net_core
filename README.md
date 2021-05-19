# angular_file_upload_net_core
Upload File or Attach file using Angular and .net core working in AWS as well

The process is to send the base64 string from Angular using POST call in the FormBody and at .net core convert the string to byte.

## At .net core

### In the controller:
```

[HttpPost("uploaddocument"), DisableRequestSizeLimit]
public IActionResult UploadDocument([FromBody] FileModel fileModel)
        {
            try
            {
                    var value = fileModel.Data;
                    var byteArray = Convert.FromBase64String(value);
                    var fileExtension = Path.GetExtension(fileModel.Name);
                    var fileName = fileModel.Name;
                    
                    /// Now save in Database or AWS

                }
                return Ok();
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



// Below Action we can take if we need to save the byte array in the Angular and use later like Multiple Attachments to send mail
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
        
```
Convert.FromBase64String(value) will convert the base64 string to byte

## In Angular Application

### HTML code:
```
<form class="row">
    <input id="{{id}}" type="file" (change)="FileSelected($event.target.files);">
</form>
<div id="hidden_upload_item" style="display: none;"></div>

```
hidden_upload_item div will be used to hold the BAse64 string temporarily

### Typescript Code:
```
FileSelected(files){
    this.fileData = <File>files[0];
    this.FileName = this.fileData.name;
    let reader = new FileReader();
    var selectedFile = <File>files[0];

    reader.onload = function (readerEvt: any) {
        var arrayBuffer = readerEvt.target.result.toString().split('base64,')[1];     
        document.querySelector('#hidden_upload_item').innerHTML = arrayBuffer;
        this.Proceed();
    }.bind(this);
    reader.readAsDataURL(selectedFile);
    
  }

  Proceed()
  {
    var item = document.querySelector('#hidden_upload_item').innerHTML;
    let formdata = new UploadFile();
    formdata.data = item;
    formdata.name = this.FileName;
    this.UploadDocument(formdata)
      .subscribe(event => {
        // Perform other operations
      }, err => {
        this.InProgress = false;
      });
  }
  
  UploadDocument(formData){
        const httpOptions = {
          headers: new HttpHeaders({
            'Content-Type': 'application/json'
          })
        };
        return this.http.post('/api/document/uploaddocument', formData, httpOptions);
      } 

```
We can call the Proceed function on some other button call. That's why I have seperated it.
### Notice: .bind(this) is used to access the component's other methods.



If you are using ConvertFileInByte in Angular 11, then the return type in the typescript is mentioned below -

```
@Injectable({
  providedIn: 'root'
})
export class CommonService {

  constructor(private http: HttpClient) { }
  ConvertFileInByte(data: any) {
    return this.http.post<FileInByteType>('Common/ConvertFileInByte', data);
  }
}

export class FileInByteType{
  fileName: string;
  data: any[];
}

```
