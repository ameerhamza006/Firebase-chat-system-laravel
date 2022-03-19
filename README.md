# Firebase-chat-system-laravel
Chat system in laravel with firebase




## Installation

1.The Firebase Admin PHP SDK is available on Packagist as [`kreait/firebase-php`](https://packagist.org/packages/kreait/firebase-php):

```bash
composer require kreait/firebase-php
```


2.Then Create Firebase Application From  your Email Account

3.Create _Realtime Database_ in firbase Application And Copy Database Url save to your NotePade

4.go to Project Setting _Service Accounts_ . _Ganerate New Privet Key_ 

5.And _Ganerate New Privet Key_ Upload To Your Controller Folder 

6.Go to  _YouProject/vendor/kreait/firebase-php/src/Firebase/Factory.php_ And Replace This Code:

On Top:
```bash
use  GuzzleHttp\Psr7\Utils;
```

Before:
```bash

    public function withDatabaseUri($uri): self
    {
        $factory = clone $this;
        $factory->databaseUri = uri_for($uri);

        return $factory;
    }
```

After:
```bash

    public function withDatabaseUri($uri): self
    {
        $factory = clone $this;
        $factory->databaseUri =  Utils::uriFor($uri);

        return $factory;
    }
```


7.Go to  Your Controller:

On Top:
```bash
use Kreait\Firebase;
use Kreait\Firebase\Factory;
use Kreait\Firebase\ServiceAccount;
use Kreait\Firebase\Database;
```

Chat Page View Function:
```bash
public function index()
{
  return view('admin.pages.chat.chats');
}
```

In Send Message Function:
```bash
public function chatsend(Request $request)
{
  if($request->message == null){
      dd($request);
   }  
         
$name = Auth::user()->name;               
$u = User::where('id',$request->receiver_id)->first();
        
$key = 'CitizenExpress'; // Your Unique Key Anything You Want
$serviceAccount = ServiceAccount::fromJsonFile(__DIR__.'/ecommercechats-firebase-adminsdk-dkxlx-fa1606dc5f.json'); //Here Is Your Generate Key file
$firebase = (new Factory)
->withServiceAccount($serviceAccount)
->withDatabaseUri('https://ecommercechats-default-rtdb.firebaseio.com') //Here Your Real Time Database Url
->create();
$database = $firebase->getDatabase();

$newPost = $database
->getReference('/messages/'.$key)
->push([
'sender_id' => Auth::user()->id,
'receiver_id' => $request->receiver_id,
'sender_name' => $name,
'receiver_name' => $u->name,
'message' => $request->message,
'date' => now()
]);


//$newPost->getKey(); // => -KVr5eu8gcTv7_AHb-3-
//$newPost->getUri(); // => https://my-project.firebaseio.com/blog/posts/-KVr5eu8gcTv7_AHb-3-
//$newPost->getChild('title')->set('Changed post title');
//$newPost->getValue(); // Fetches the data from the realtime database
//$newPost->remove();
//echo"<pre>";
//print_r($newPost->getvalue());
  
$output = '';
       
foreach($database->getReference('/messages/CitizenExpress')->getValue() as $v){
   
if($v['sender_id'] == $request->receiver_id ){
                     
$output .= '<div class="direct-chat-msg">
<div class="direct-chat-infos clearfix">
<span class="direct-chat-name float-left">'.$v['sender_name'].'</span>
<span class="direct-chat-timestamp float-right">'.date('d-m-Y', strtotime($v['date'])).'</span>
</div>
<div class="direct-chat-text" style="margin: 0px;" >';                  
$output .= '    '.$v['message'].' ';
$output .= ' </div></div>';
  
    }
    
if($v['sender_id'] == Auth::user()->id){
   
if($v['receiver_id'] == $request->receiver_id){
                                    
$output .= '<div class="direct-chat-msg right" >
<div class="direct-chat-infos clearfix">
<span class="direct-chat-name float-right">You</span>
<span class="direct-chat-timestamp float-left">'.date('d-m-Y', strtotime($v['date'])).'</span>
</div>
<div class="direct-chat-text"  style="margin: 0px;">';
$output .= '    '.$v['message'].' ';
$output .= ' </div></div>';                                 
   }
                                    
    }
}
       
         
return $output;
       
    }
```


In See All Message Function:
```bash
public function chatsall(Request $request)
{
  if($request->message == null){
      dd($request);
   }  
         
$name = Auth::user()->name;               
$u = User::where('id',$request->receiver_id)->first();
        
$key = 'CitizenExpress'; // Your Unique Key Anything You Want
$serviceAccount = ServiceAccount::fromJsonFile(__DIR__.'/ecommercechats-firebase-adminsdk-dkxlx-fa1606dc5f.json'); //Here Is Your Generate Key file
$firebase = (new Factory)
->withServiceAccount($serviceAccount)
->withDatabaseUri('https://ecommercechats-default-rtdb.firebaseio.com') //Here Your Real Time Database Url
->create();
$database = $firebase->getDatabase();



$output = '';
       
foreach($database->getReference('/messages/CitizenExpress')->getValue() as $v){
   
if($v['sender_id'] == $request->receiver_id ){
                     
$output .= '<div class="direct-chat-msg">
<div class="direct-chat-infos clearfix">
<span class="direct-chat-name float-left">'.$v['sender_name'].'</span>
<span class="direct-chat-timestamp float-right">'.date('d-m-Y', strtotime($v['date'])).'</span>
</div>
<div class="direct-chat-text" style="margin: 0px;" >';                  
$output .= '    '.$v['message'].' ';
$output .= ' </div></div>';
  
    }
    
if($v['sender_id'] == Auth::user()->id){
   
if($v['receiver_id'] == $request->receiver_id){
                                    
$output .= '<div class="direct-chat-msg right" >
<div class="direct-chat-infos clearfix">
<span class="direct-chat-name float-right">You</span>
<span class="direct-chat-timestamp float-left">'.date('d-m-Y', strtotime($v['date'])).'</span>
</div>
<div class="direct-chat-text"  style="margin: 0px;">';
$output .= '    '.$v['message'].' ';
$output .= ' </div></div>';                                 
   }
                                    
    }
}
       
         
return $output;
       
    }
```

8. Create Routes
```bash
Route::get('chat',[ChatController::class,'index'])->name('index');
Route::post('chat/send', [ChatController::class, 'chatsend'])->name('chat.send');
Route::get('chat/all', [ChatController::class, 'chatsall'])->name('chat.all');
```

9.Go to Your Chat View Page:

Chat Box Where Chat Send And Show:
```bash

<a href="/chat?user=1" class="nav-link">Ameer Hamza</a> <!---- In Your User List with Send Id To Url like This ------>

          <div class="col-md-9">
              @php
               if(isset($_GET['user'])){ // Get reciver id From Url  
              $id = $_GET['user'];
             
              $userr = \App\Models\User::where('id',$id)->first();
              
              
              }else{
              $id = '';
              }
             
              @endphp
            <!-- DIRECT CHAT PRIMARY -->
            <div class="card card-primary card-outline direct-chat direct-chat-primary" style=" height: 500px;">
              <div class="card-header">
                <h3 class="card-title"> @if($id != '')  {{$userr->name}} @else   @endif</h3>
                @if($id != '')
                
                    &nbsp;<i class="far fa-circle text-success"  style=" background-color: #28a745!important; border-radius: 20px;" ></i>
                @endif

               
              </div>
              <!-- /.card-header -->
              <div class="card-body">
                <!-- Conversations are loaded here -->
           
                
                <div class="direct-chat-messages" id="allmessaages" style="height:100%;" >
                    
                 
                </div>
                
                <!--/.direct-chat-messages-->

                <!-- Contacts are loaded here -->
                <div class="direct-chat-contacts">
                  <ul class="contacts-list">
                    <li>
                      <a href="#">
                        <img class="contacts-list-img" src="{{asset('admin/dist/img/user1-128x128.jpg')}}" alt="User Avatar">

                        <div class="contacts-list-info">
                          <span class="contacts-list-name">
                            Count Dracula
                            <small class="contacts-list-date float-right">2/28/2015</small>
                          </span>
                          <span class="contacts-list-msg">How have you been? I was...</span>
                        </div>
                        <!-- /.contacts-list-info -->
                      </a>
                    </li>
                    <!-- End Contact Item -->
                  </ul>
                  <!-- /.contatcts-list -->
                </div>
                <!-- /.direct-chat-pane -->
              </div>
              <!-- /.card-body -->
              
              <div class="card-footer">
                  <form id="data" method="post" enctype="multipart/form-data">
						    @csrf
						     <input type="hidden" name="sender_id" value="{{Auth::user()->id}}" />
						     @if($id != '')
						     <input type="hidden" name="receiver_id" value="{{$id}}" />
						     
						 
                  <div class="input-group">
                    <input type="text" id="sendchat" name="message"  placeholder="Type Message ..." autocomplete="off" class="form-control sendchat">
                      
                  <!----<div class="btn btn-default btn-file">
                    <i class="fas fa-paperclip"></i> Attachment
                    <input type="file" name="file">
                 
                </div> -->
                
                    <span class="input-group-append">
                      <button type="submit"  class="btn btn-primary">Send</button>
                    </span>
                  </div>
                  @endif
                </form>
              </div>
           
              <!-- /.card-footer-->
            </div>
            <!--/.direct-chat -->
          </div>

```


10. JavaScript:

```bash
<script src="https://code.jquery.com/jquery-3.5.0.js"></script>
<script src="https://ajax.googleapis.com/ajax/libs/jquery/3.1.0/jquery.min.js"></script>

<script>
	
 $(document).ready(function(){
 fetch();

	
 $("#allmessaages").animate({ scrollTop: 20000000 }, "slow");
});
    
 setInterval(function(){
 
  fetch();
 }, 5000);
 
 
 function fetch(){
        
    $.ajax({
     url:"{{route('chat.all')}}",
    method:"GET",
    data:{receiver_id:"{{$id}}"},
    success:function(result){
    $('#allmessaages').html(result);
     }
     })
        
    }
    
    $("form#data").submit(function(e) {
    e.preventDefault();    
    var formData = new FormData(this);
    //console.log(formData)
    $.ajax({
        url: "{{route('chat.send')}}",
        type: 'POST',
        data: formData,
        success: function (result) {
               
         $("input[type='text']").val("");
         $("input[type='file']").val("");
              
         $('#allmessaages').html(result);
        },
        cache: false,
        contentType: false,
        processData: false
    });
});
    


</script>

```
