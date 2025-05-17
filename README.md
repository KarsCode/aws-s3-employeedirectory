# aws-s3-employeedirectory
This project replicates a hands-on AWS lab where an EC2-hosted employee directory application is configured to serve employee images from an S3 bucket. The app leverages IAM roles for secure access to S3.

## Objectives
- Create an Amazon Simple Storage Service (Amazon S3) Bucket
- Create an S3 bucket policy
- Modify an Application to Use an S3 Bucket
- Upload an Object to an S3 Bucket
- Create an Amazon DynamoDB table
- Test an Application Using an Application Web Interface
- Manage Existing DynamoDB Items Using the AWS Management Console
- Create Items in a DynamoDB Table Using the AWS Management Console

## Task 1 : Create an Amazon Simple Storage Service (Amazon S3) Bucket

Existing architecture I will be working with is described as follows: 

![image](https://us-west-2-tcprod.s3.us-west-2.amazonaws.com/courses/SPL-CX-100-CETEWA/v1.0.1.prod-b4ef746d/instructions/en_us/images/lab-3-overview.png)

 #### Application Context

The Employee Directory app was already deployed within Public Subnet 1 of a preconfigured VPC. Although the app was publicly accessible via a provided URL, it initially displayed an error:

- S3: Employee Images bucket not found.
- DynamoDB: ‘Employees’ table not found.

This indicated that no S3 bucket was linked for hosting image assets, and the application lacked the necessary IAM permissions to interact with S3.


#### Steps Performed

    Created a New S3 Bucket
    I navigated to the Amazon S3 console and created a new bucket with the following configuration:

        Bucket Name: employee-photo-bucket-<my-initials>-<4-digit-random-number> (e.g., employee-photo-bucket-al-2837)

        Region: Matched the AWS region in which the application was deployed

        Block Public Access: Kept the default setting (Block all public access) enabled to ensure secure access

        Note: Every S3 bucket name must be globally unique. I appended my initials and a random number to avoid naming conflicts.

    Verified Bucket Creation
    Once the bucket was created, AWS confirmed with a success message, and the new bucket became visible in the S3 console.

#### IAM Role Context

The application uses an IAM role named EmployeeDirectoryAppRole, which is attached to the EC2 instance hosting the app. This role hadn’t yet been granted access to the S3 bucket — a requirement for the application to load image assets via signed AWS API requests.

## Task 2 : Create an S3 Bucket Policy

#### Opened the S3 Bucket

I navigated to the S3 Management Console and selected the bucket I created earlier:

```
employee-photo-bucket-<my-initials>-<random-number>
```

#### Navigated to the Permissions Tab

Once inside the bucket view, I switched to the **Permissions** tab and located the **Bucket policy** section. I clicked **Edit** to open the JSON policy editor.

#### Created the Bucket Policy

I manually pasted the following policy into the editor:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowS3ReadAccess",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::000000000000:role/EmployeeDirectoryAppRole"
      },
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::INSERT_BUCKET_NAME",
        "arn:aws:s3:::INSERT_BUCKET_NAME/*"
      ]
    }
  ]
}
```

I replaced:
- `000000000000` with the **AWS Account ID** provided in the lab instructions.
- `INSERT_BUCKET_NAME` with my actual bucket name.

#### Saved the Policy

Before saving, I made sure there were no leading whitespaces before the first `{`. Then I clicked **Save changes**, and saw a success message:

```
Successfully edited bucket policy
```
This confirmed that my policy was accepted and attached to the bucket.

The IAM role `EmployeeDirectoryAppRole` can now access all objects in the S3 bucket. This means my Employee Directory application can now fetch and display employee images from Amazon S3 without storing credentials on the EC2 instance.

This step was essential to establishing **secure and programmatic access** between the compute and storage layers of my AWS-hosted application.


## Task 3 : Modify the Application to Use the S3 Bucket

I now configure the application to use the bucket as a source of employee images
#### Opened the Employee Directory Web Application

I returned to the **Employee Directory** application by navigating to the web URL provided in the lab environment.

#### Accessed the Configuration Settings

Inside the web interface, I scrolled down to the **Administration** section and selected **Configuration**.

At this point, the configuration showed:

- **S3 Access Enabled**: `false`
- **S3 Bucket**: *(blank)*

This meant that no bucket had been associated yet.

![image](https://us-west-2-tcprod.s3.us-west-2.amazonaws.com/courses/SPL-CX-100-CETEWA/v1.0.1.prod-b4ef746d/instructions/en_us/images/lab-3-app-config.png)

#### Linked My S3 Bucket

I clicked **Change** next to the S3 Bucket field, then entered the name of the bucket I had previously created:

```
employee-photo-bucket-ks-8181
```

I clicked **Save** to confirm the change.

#### Verified the Configuration

After saving, the Configuration Settings updated to:

- **S3 Access Enabled**: `true`
- **S3 Bucket**: `employee-photo-bucket-ks-8181`

This confirmed that the Employee Directory application was now successfully configured to retrieve employee images from the specified S3 bucket.

The application now knows where to look for image data, and the S3 access is properly enabled through IAM role-based permissions and linked configuration. At this stage, while no images are in the bucket yet, the groundwork is set for dynamic image loading from Amazon S3.

## Task 4 : Upload an Object to an S3 Bucket

After setting up my S3 bucket and granting the correct IAM permissions to my EC2 instance, the next step was to upload image objects to the bucket. Objects in S3 can be any type of file, such as text, images, videos, or archives. For this project, I uploaded a set of `.png` images that represent employee profile photos.

#### Downloading the Image Files

I started by copying the `PhotosZipURL` provided in the lab instructions. I opened this URL in a new browser tab, and it immediately downloaded a `.zip` file containing 10 sample employee images.

Once the download was complete, I extracted the compressed archive to a directory on my local computer. After extraction, I confirmed that the folder contained 10 `.png` files.

#### Uploading to S3

Next, I returned to the AWS Management Console and navigated back to my S3 bucket.

From the **Objects** tab inside the bucket, I selected **Upload**. In the **Files and folders** section, I clicked **Add files** and selected all 10 `.png` images from my local directory.

Once the files appeared in the upload list, I scrolled to the bottom of the page and chose **Upload**.

After a few seconds, I saw a confirmation message saying:

```
Upload succeeded
```

I then clicked **Close** to return to the bucket’s object list and verified that all 10 images had been uploaded successfully.

#### Verifying in the Web Application

To verify that everything worked, I went back to the **Employee Directory** web application. In the **Employees** section, I clicked on **Images**.

![image](https://us-west-2-tcprod.s3.us-west-2.amazonaws.com/courses/SPL-CX-100-CETEWA/v1.0.1.prod-b4ef746d/instructions/en_us/images/lab-3-employee-images.png)

The application displayed the uploaded employee images exactly as expected. This confirmed that the IAM role had the necessary permissions and the S3 bucket was correctly configured.


## Task 5 : Create an Amazon DynamoDB Table

Now that the S3 configuration is complete, the employee directory application is almost ready. But first, I must configure the application to present employee data using Amazon DynamoDB.
#### Navigating to DynamoDB

At the top of the AWS Management Console, I used the search bar to look for **DynamoDB**. Once it appeared in the search results, I selected it to enter the DynamoDB dashboard.

#### Creating the Table

Inside the DynamoDB console, I chose **Create table**. Depending on the screen size, the button might not be immediately visible, so I made sure to click **Tables** from the left-hand navigation pane if needed, and then clicked **Create table**.

I filled out the following fields exactly as specified:

- **Table name**: `Employees`  
- **Partition key**: `id`  
- **Key type**: `String`  

I made sure **not** to modify any other settings on the page.

Once I was done, I scrolled to the bottom and clicked **Create table**.

#### Waiting for Table Creation

It took a moment for the table to be created. I monitored the status indicator next to the new table entry, and once it showed **Active**, I knew the table was ready for use.

## Task 6 : Testing the Application using the Application Web Interface

With the S3 bucket and DynamoDB table now fully integrated into the employee directory application, it was time to test everything using the web interface.

#### Opening the Application

I returned to the browser tab where the employee directory application was open and refreshed the page. If the tab had been closed, I would have opened a new one and pasted the value from **PublicWebApplicationURL** into the address bar to access the app.

#### Verifying DynamoDB Integration

From the application interface, I navigated to the **Administration** section on the left-hand side and selected **Configuration**. 

I noticed that the previous DynamoDB error was now gone — this confirmed that the IAM role attached to my EC2 instance had the necessary permissions and that the application could now successfully interact with the DynamoDB table.

#### Adding a New Employee

To test the full workflow, I did the following:

1. I located the **Employees** section on the left pane and chose **Management**.
2. From the **Actions** dropdown, I selected **New Employee**.
3. In the form that appeared, I entered:
   - **Name**: Jane Doe
   - **Location**: New York
   - **Email**: jane.doe@example.com
4. Under **Select existing photo**, I used the **Photo** dropdown to choose one of the uploaded images.
5. I scrolled to the bottom and clicked **Add**.

The application created a new record instantly, and it appeared in the employee list — pulling data from the DynamoDB table and displaying the associated S3 image.

![image](https://us-west-2-tcprod.s3.us-west-2.amazonaws.com/courses/SPL-CX-100-CETEWA/v1.0.1.prod-b4ef746d/instructions/en_us/images/lab-3-task-6-add-employee.png)

#### Exploring Further

To get more hands-on experience, I added a couple more employee records, edited an existing one, and even deleted a few. Each operation was successfully reflected in the application and backed by updates to the underlying DynamoDB table.


## Task 7 : Manage Existing DynamoDB Items Using the AWS Management Console

With the employee directory application fully functional, I wanted to validate that the employee data I added through the app was indeed stored in my DynamoDB table. In this task, I explored, edited, and verified data directly from the AWS Management Console.

#### Accessing the DynamoDB Table

I navigated back to the browser tab where my DynamoDB table was open. Under the **Tables** section, I selected the **Employees** link to view the table’s details.

I spent a moment reviewing the **Overview** and **Indexes** tabs to get familiar with the metadata and indexing structure of the table.

#### Exploring Table Items

To view the actual employee records stored in the table, I chose **Explore table items**. This brought up a list of all the current items, which were originally created via the web application.

From the list, I selected one item by clicking on the value in the **id** column. This opened the detailed view of the record.

#### Editing the Item

The **Edit item** page displayed all the attributes of the selected employee. At the top, I had two view options: a structured view and a **JSON** view. I switched to the **JSON** tab to make direct edits.

I carefully updated the `location` and `email` fields. I changed the `photo` field too, but I ensured that any new value matched the filename of an existing object in my S3 bucket.

**Note**: I made sure not to touch the `id` field. Since it’s the partition key, editing it would result in an error. If I ever need to change it, I’ll have to delete and recreate the item.

Once I was satisfied with the edits, I clicked **Save changes**.

#### Verifying Changes in the Web App

To confirm that the changes reflected correctly, I went back to the browser tab with the employee directory application and refreshed the page. The updated information showed up immediately — a clear sign that the app was successfully pulling live data from DynamoDB

## Task 8 : Create Items in a DynamoDB Table Using the AWS Management Console

In this task, I manually created new employee records in my DynamoDB table using the AWS Management Console. This was a good opportunity to reinforce my understanding of how individual attributes are added and how changes propagate to the connected web application.

#### Creating a New Item

I started by going to the browser tab with my DynamoDB table open. In the **Items returned** section, I clicked **Create item** to begin.

I made sure to use the **Form view** to easily add and structure each attribute.

First, I provided a unique value for the **id** field. Since this is the partition key, I ensured it didn’t duplicate any existing `id` values in the table.

Then I proceeded to add attributes one by one:

- I clicked **Add new attribute**, selected `String`, replaced `NewValue` with `name`, and gave it a value like `John`.
- I repeated the process for `location`, setting it to `New York`.
- I added `email`, using a value such as `john.doe@example.org`.
- Finally, I added the `photo` attribute as a `String`, but left it **empty** for now.

Once all the required fields were added, I clicked **Create item**. The console confirmed:

> *The item has been saved successfully.*

#### Verifying the New Record in the Web App

To check that everything was working, I returned to the employee directory web application in my browser and refreshed the page.

The newly added employee record appeared in the list right away, confirming that the connection between DynamoDB and the application backend was functioning correctly.

#### Additional Hands-on Practice

For more practice, I created a few more employee records manually using the same steps. I also experimented with editing and deleting items, ensuring each one included all the required fields: `name`, `location`, `email`, and `photo`.

Eventually, I started assigning real values to the `photo` attribute, using filenames that matched the objects I had uploaded earlier into my S3 bucket.

