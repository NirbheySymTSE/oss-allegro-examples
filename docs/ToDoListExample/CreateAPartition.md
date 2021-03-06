---
nav_order: 18
parent: ToDo List Example
---
# Create A Partition

The Create Partition example creates a new Partition.

The first part of the program is almost identical to the [Hello World Example](/HelloWorld.html), see the description
there for more details.

In this case we also need to provide the Object Store URL, since these examples use that API.

```java
public class CreatePartition extends CommandLineHandler implements Runnable
{
  private static final String ALLEGRO         = "ALLEGRO_";
  private static final String SERVICE_ACCOUNT = "SERVICE_ACCOUNT";
  private static final String POD_URL         = "POD_URL";
  private static final String OBJECT_STORE_URL = "OBJECT_STORE_URL";
  private static final String CREDENTIAL_FILE = "CREDENTIAL_FILE";
  private static final String OTHER_USER_ID    = "OTHER_USER_ID";
  
  private String              serviceAccount_;
  private String              podUrl_;
  private String              objectStoreUrl_;
  private String              credentialFile_;
  private PodAndUserId        otherUserId_;
  
  private IAllegroApi         allegroApi_;

  /**
   * Constructor.
   */
  public CreatePartition()
  {
    withFlag('s',   SERVICE_ACCOUNT,  ALLEGRO + SERVICE_ACCOUNT,  String.class,   false, true,   (v) -> serviceAccount_       = v);
    withFlag('p',   POD_URL,          ALLEGRO + POD_URL,          String.class,   false, true,   (v) -> podUrl_               = v);
    withFlag('o',   OBJECT_STORE_URL, ALLEGRO + OBJECT_STORE_URL, String.class,   false, true,   (v) -> objectStoreUrl_       = v);
    withFlag('f',   CREDENTIAL_FILE,  ALLEGRO + CREDENTIAL_FILE,  String.class,   false, false,  (v) -> credentialFile_       = v);
    withFlag(null,  OTHER_USER_ID,    ALLEGRO + OTHER_USER_ID,    Long.class,     false, false,  (v) -> otherUserId_          = PodAndUserId.newBuilder().build(v));
  }
  
  @Override
  public void run()
  { 
    allegroApi_ = new AllegroApi.Builder()
      .withPodUrl(podUrl_)
      .withObjectStoreUrl(objectStoreUrl_)
      .withUserName(serviceAccount_)
      .withRsaPemCredentialFile(credentialFile_)
      .withTrustAllSslCerts()
      .build();
    
    System.out.println("PodId is " + allegroApi_.getPodId());
    
```

In the next part of the program, if an additional userId has been provided,
we create a ResourcePermissions object which grants access to the new Partition for that other user. 
Note that this userId based access control will be supplemented with an entitlement based system soon.

```java
    ResourcePermissions permissions = null;
    
    if(otherUserId_ != null)
    {
      permissions = new ResourcePermissions.Builder()
          .withUser(otherUserId_, Permission.Read, Permission.Write)
          .build()
          ;
    }
```

Next we create the Partition, note that this is an _upsert_ operation and if the partition already exists the effect
is simply to add the given ResourcePermissions. If they also exist then this operation is a NO OP.

In order to create an Object with an ID which we can look up, we need to create an ID object, which is an unencrypted
object containing some identifier which is known externally. For security reasons (to prevent certain types
of denial of service attack) all ID objects created through Allegro _must_ include the user ID of the user
who created the ID object.

The __UpsertPartitionRequest__ object allows an ID to be specified directly, that ID object can contain whatever set
of attributes the application needs to be able to look up the Partition in future. As a convenience, the API provides
an ID object type called __NamedUserIdObject__ which contains a userId and a String name attribute. In cases where
a simple name is sufficient the name can be passed on its own, as we do here:

```java
    IPartition partition = allegroApi_.upsertPartition(new UpsertPartitionRequest.Builder()
          .withName(CalendarApp.PARTITION_NAME)
          .withPermissions(permissions)
          .build()
        );
    
    System.out.println("partition is " + partition);
```

When we run the program we see the following output:

```

2020-06-09T07:27:37.120000000Z [main               ] INFO  com.symphony.oss.allegro.api.AllegroApi - AllegroApi constructor start
2020-06-09T07:27:37.465000000Z [main               ] INFO  com.symphony.oss.allegro.api.AllegroApi - sbe auth....
2020-06-09T07:27:38.207000000Z [main               ] INFO  com.symphony.oss.allegro.api.AllegroApi - fetch podInfo_....
2020-06-09T07:27:38.331000000Z [main               ] INFO  com.symphony.oss.allegro.api.AllegroApi - keymanager auth....
2020-06-09T07:27:38.501000000Z [main               ] INFO  com.symphony.oss.allegro.api.AllegroApi - getAccountInfo....
2020-06-09T07:27:38.857000000Z [main               ] INFO  com.symphony.oss.allegro.api.AllegroApi - userId_ = 351775001412007
2020-06-09T07:27:38.864000000Z [main               ] INFO  com.symphony.oss.allegro.api.AllegroApi - allegroApi constructor done.
PodId is 5119
partition is {
  "_type":"com.symphony.s2.model.object.Partition",
  "_version":"1.0",
  "hashType":1,
  "id":{
    "_type":"com.symphony.s2.model.object.NamedUserIdObject",
    "_version":"1.0",
    "hashType":1,
    "name":"MyCalendar",
    "userId":351775001412007
  }
}
```


