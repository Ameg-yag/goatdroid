# goatdroid
#Source for GoatDroid trying to exploit XSS --> this is the ViewCheckin activity.
  package org.owasp.goatdroid.fourgoats.activities;

  import android.content.Context;
  import android.content.Intent;
  import android.os.AsyncTask;
  import android.os.Bundle;
  import android.webkit.WebSettings;
  import android.webkit.WebView;
  import java.util.HashMap;
  import org.owasp.goatdroid.fourgoats.base.BaseActivity;
  import org.owasp.goatdroid.fourgoats.db.UserInfoDBHelper;
  import org.owasp.goatdroid.fourgoats.javascriptinterfaces.SmsJSInterface;
  import org.owasp.goatdroid.fourgoats.javascriptinterfaces.ViewCheckinJSInterface;
  import org.owasp.goatdroid.fourgoats.javascriptinterfaces.WebViewJSInterface;
  import org.owasp.goatdroid.fourgoats.misc.Utils;
  import org.owasp.goatdroid.fourgoats.rest.viewcheckin.ViewCheckinRequest;

  public class ViewCheckin
    extends BaseActivity
  {
    Bundle bundle;
    Context context;
    String sessionToken;
    WebView webview;
  
    public String generateComments(HashMap<String, String> paramHashMap)
    {
      String str1 = "";
      int i;
      int j;
      if (paramHashMap.size() > 3)
      {
        if (paramHashMap.size() / 6 - 3 != -2) {
          break label60;
        }
      i = 1;
      str1 = "" + "<b><big>Comments:</big></b><p>";
      j = 0;
    }
    for (;;)
    {
      if (j >= i)
      {
        return str1;
        label60:
        i = paramHashMap.size() / 6;
        break;
      }
      String str2 = (String)paramHashMap.get("firstName" + j);
      String str3 = (String)paramHashMap.get("lastName" + j);
      Object localObject = ((String)paramHashMap.get("dateTime" + j)).split(" ");
      String str4 = localObject[0];
      localObject = localObject[1];
      String str5 = (String)paramHashMap.get("comment" + j);
      str1 = new StringBuilder(String.valueOf(new StringBuilder(String.valueOf(str1)).append("<p><b>").append(str2).append(" ").append(str3).append("</b><br>").append(str4).append("<br>").append((String)localObject).append("<br>").toString())).append("<b>\"").append(str5).append("\"</b><br>").toString() + "<button style=\"color: white; background-color:#2E9AFE\" type=\"button\" onclick=\"window.viewCheckinJSInterface.deleteComment('" + (String)paramHashMap.get(new StringBuilder("commentID").append(j).toString()) + "','" + this.bundle.getString("venueName") + "','" + this.bundle.getString("venueWebsite") + "','" + this.bundle.getString("dateTime") + "','" + this.bundle.getString("latitude") + "','" + this.bundle.getString("longitude") + "','" + this.bundle.getString("checkinID") + "')\">" + "Delete Comment</button><br>";
      j += 1;
    }
    }
  
    public String generateViewCheckinHTML(HashMap<String, String> paramHashMap)
    {
      String str = "" + "<p><b>" + this.bundle.getString("venueName") + "</b></p>";
      String[] arrayOfString = this.bundle.getString("dateTime").split(" ");
      return new StringBuilder(String.valueOf(new StringBuilder(String.valueOf(new StringBuilder(String.valueOf(new   StringBuilder(String.valueOf(str)).append("<p><b>Date:</b> ").append(arrayOfString[0]).append(" <b>Time:</b> ").append(arrayOfString[1]).append("</p>").toString())).append("<button style=\"color: white; background-color:#2E9AFE\" type=\"button\" onclick=\"window.webViewJSInterface.launchWebView('").append(this.bundle.getString("venueWebsite")).append("')\">").append("Visit Website</button><br><br>").toString())).append("<button style=\"color: white; background-color:#2E9AFE\" type=\"button\" onclick=\"window.smsJSInterface.launchSendSMSActivity('").append(this.bundle.getString("venueName")).append("','").append(this.bundle.getString("dateTime")).append("')\">").append("Text This To A Friend</button><p>").toString())).append("<button style=\"color: white; background-color:#2E9AFE\" type=\"button\" onclick=\"window.viewCheckinJSInterface.launchDoCommentActivity('").append(this.bundle.getString("venueName")).append("','").append(this.bundle.getString("venueWebsite")).append("','").append(this.bundle.getString("dateTime")).append("','").append(this.bundle.getString("latitude")).append("','").append(this.bundle.getString("longitude")).append("','").append(this.bundle.getString("checkinID")).append("')\">").append("Leave a Comment</button><br><br>").toString() + generateComments(paramHashMap);
    }
  
    public void launchLogin()
    {
      startActivity(new Intent(this.context, Login.class));
    }
  
    public void onCreate(Bundle paramBundle)
    {
      super.onCreate(paramBundle);
      setContentView(2130903097);
      this.context = getApplicationContext();
      this.bundle = getIntent().getExtras();
      this.webview = ((WebView)findViewById(2130968678));
      this.webview.getSettings().setJavaScriptEnabled(true);
      this.webview.addJavascriptInterface(new SmsJSInterface(this), "smsJSInterface");
      this.webview.addJavascriptInterface(new ViewCheckinJSInterface(this), "viewCheckinJSInterface");
      this.webview.addJavascriptInterface(new WebViewJSInterface(this), "webViewJSInterface");
      new GetCommentData(null).execute(new Void[] { null, null });
    }
  
    private class GetCommentData
      extends AsyncTask<Void, Void, HashMap<String, String>>
    {
      private GetCommentData() {}
    
      protected HashMap<String, String> doInBackground(Void... paramVarArgs)
      {
        paramVarArgs = new HashMap();
        UserInfoDBHelper localUserInfoDBHelper = new UserInfoDBHelper(ViewCheckin.this.context);
        Object localObject = localUserInfoDBHelper.getSessionToken();
        ViewCheckinRequest localViewCheckinRequest = new ViewCheckinRequest(ViewCheckin.this.context);
        try
        {
          localObject = localViewCheckinRequest.getCheckin((String)localObject, ViewCheckin.this.bundle.getString("checkinID"));
          paramVarArgs = (Void[])localObject;
          if (((String)((HashMap)localObject).get("success")).equals("true"))
          {
            paramVarArgs = (Void[])localObject;
            ((HashMap)localObject).put("htmlResponse", ViewCheckin.this.generateViewCheckinHTML((HashMap)localObject));
          }
          return (HashMap<String, String>)localObject;
        }
        catch (Exception localException)
        {
          paramVarArgs.put("errors", localException.getMessage());
          paramVarArgs.put("success", "false");
          return paramVarArgs;
        }
        finally
        {
          localUserInfoDBHelper.close();
        }
      }
    
      public void onPostExecute(HashMap<String, String> paramHashMap)
      {
        if (((String)paramHashMap.get("success")).equals("true"))
        {
          ViewCheckin.this.webview.loadData((String)paramHashMap.get("htmlResponse"), "text/html", "UTF-8");
          return;
        }
        if (((String)paramHashMap.get("errors")).equals("Invalid session"))
        {
          ViewCheckin.this.launchLogin();
          return;
        }
        Utils.makeToast(ViewCheckin.this.context, (String)paramHashMap.get("errors"), 1);
      }
    }
    }












#This is the ViewCheckin JSInterface

  package org.owasp.goatdroid.fourgoats.javascriptinterfaces;

  import android.content.Context;
  import android.content.Intent;
  import android.os.Bundle;
  import java.util.HashMap;
  import org.owasp.goatdroid.fourgoats.activities.DoComment;
  import org.owasp.goatdroid.fourgoats.activities.ViewCheckin;
  import org.owasp.goatdroid.fourgoats.db.UserInfoDBHelper;
  import org.owasp.goatdroid.fourgoats.misc.Utils;
  import org.owasp.goatdroid.fourgoats.rest.comments.CommentsRequest;

  public class ViewCheckinJSInterface
  {
    Context mContext;
  
    public ViewCheckinJSInterface(Context paramContext)
    {
      this.mContext = paramContext;
    }
  
    public void deleteComment(String paramString1, String paramString2, String paramString3, String paramString4, String paramString5, String paramString6, String paramString7)
    {
      Object localObject2 = new UserInfoDBHelper(this.mContext);
      Object localObject1 = ((UserInfoDBHelper)localObject2).getSessionToken();
      ((UserInfoDBHelper)localObject2).close();
      localObject2 = new CommentsRequest(this.mContext);
      try
      {
        paramString1 = ((CommentsRequest)localObject2).removeComment((String)localObject1, paramString1);
        if (((String)paramString1.get("success")).equals("true"))
        {
          Utils.makeToast(this.mContext, "Comment has been removed!", 1);
          paramString1 = new Intent(this.mContext, ViewCheckin.class);
          localObject1 = new Bundle();
          ((Bundle)localObject1).putString("venueName", paramString2);
          ((Bundle)localObject1).putString("venueWebsite", paramString3);
          ((Bundle)localObject1).putString("dateTime", paramString4);
          ((Bundle)localObject1).putString("latitude", paramString5);
          ((Bundle)localObject1).putString("longitude", paramString6);
          ((Bundle)localObject1).putString("checkinID", paramString7);
          paramString1.putExtras((Bundle)localObject1);
          this.mContext.startActivity(paramString1);
          return;
        }
        Utils.makeToast(this.mContext, (String)paramString1.get("errors"), 1);
        return;
      }
      catch (Exception paramString1)
      {
        Utils.makeToast(this.mContext, "Something weird happened", 1);
      }
    }
  
    public void launchDoCommentActivity(String paramString1, String paramString2, String paramString3, String paramString4,   String paramString5, String paramString6)
    {
      Intent localIntent = new Intent(this.mContext, DoComment.class);
      Bundle localBundle = new Bundle();
      localBundle.putString("venueName", paramString1);
      localBundle.putString("venueWebsite", paramString2);
      localBundle.putString("dateTime", paramString3);
      localBundle.putString("latitude", paramString4);
      localBundle.putString("longitude", paramString5);
      localBundle.putString("checkinID", paramString6);
      localIntent.putExtras(localBundle);
      this.mContext.startActivity(localIntent);
    }
  
    public void launchViewCheckinActivity(String paramString1, String paramString2, String paramString3, String paramString4, String paramString5, String paramString6)
    {
      Intent localIntent = new Intent(this.mContext, ViewCheckin.class);
      Bundle localBundle = new Bundle();
      localBundle.putString("venueName", paramString1);
      localBundle.putString("venueWebsite", paramString2);
      localBundle.putString("dateTime", paramString3);
      localBundle.putString("latitude", paramString4);
      localBundle.putString("longitude", paramString5);
      localBundle.putString("checkinID", paramString6);
      localIntent.putExtras(localBundle);
      this.mContext.startActivity(localIntent);
    }
  }
