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

5.Go to  _YouProject/vendor/kreait/firebase-php/src/Firebase/Factory.php_ And Replace This Code:

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


6.Go to  Your Controller:

On Top:
```bash
use Kreait\Firebase;
use Kreait\Firebase\Factory;
use Kreait\Firebase\ServiceAccount;
use Kreait\Firebase\Database;
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
public function chatall(Request $request)
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

