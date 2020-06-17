---
title: Controlling data access - Leveraving PowerBI RLS with PowerApps
author: Oscar Toro
date: '2020-06-16'
slug: controlling-data-access-leveraving-powerbi-rls-with-powerapps
categories:
  - PowerBI
  - PowerApps
tags:
  - RLS
  - Row Level Security
  - PowerApps
  - PowerBI
  - Data
  - Report
  - Dashboard
subtitle: ''
summary: ''
authors: []
lastmod: '2020-06-16T12:32:42+10:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---

## Introduction

From experience, managing who and who can't access data can become quite a sensitive topic, even within organizations. Having security requirements might led to creating multiple reports or to maintain excel spreadsheets which might be prone to human error.

In this article I will be explaining how to implement RLS (row-level-security) in PowerBI reports, and how to create an easy tool to maintain it by leveraging the SharePoint infrastructure.

The steps to achieve this are:

1. Determine what can be seen by whom with PowerApps
2. PowerBI integration
3. Sharing the report

## 1. Determine what can be seen by whom

### 1.1. Set-up the structure

Our first step is to determine how we will be managing permissions, or who sees what within the report. We face the case that multiple people will be able to see multiple projects within the report, as shown in the image below.

![](/post/2020-06-16-controlling-data-access-leveraving-powerbi-rls-with-powerapps.en_files/figure1.jpg)

We have to give permission to each person to each project individually. In order to achieve this we set-up a SharePoint list that contains the fields:

- Title (Set by default), column type **text**.
- Project (Field to control visualization) column type **text**.
- User, column type **person**, do not allow the selection of groups.
- E-mail, column type **text**.
- user_text, column type **text**.
- date, column type **date**

![](/post/2020-06-16-controlling-data-access-leveraving-powerbi-rls-with-powerapps.en_files/figure2.png)

Additional control fields can be established (e.g. date the access has been given, who gave access) but they are not necessary.
Please note that people who will be giving permissions should have access to this list, so the place where this is set within SharePointim is very important.

### 1.2. PowerApp

Our next step is creating a PowerApp to populate and control this list. I will not go into much detail on how to creat PowerApps here and will stick to a very high level on how to set the PowerApp and direct the information. If you want more information on how to make PowerApps I recommend this [free Microsoft edx course.](https://www.edx.org/course/developing-business-applications-with-microsoft-po)

The final output of our PowerApp will look like this:

![](/post/2020-06-16-controlling-data-access-leveraving-powerbi-rls-with-powerapps.en_files/figure22.png)

#### 1.2.1. PowerApp elements: Form

We create a **Blank Canvas App**, and, on the main page, we include a **form** in edit mode:

![](/post/2020-06-16-controlling-data-access-leveraving-powerbi-rls-with-powerapps.en_files/figure3.png)

[Connect the form to our Sharepoint list](https://docs.microsoft.com/en-us/powerapps/maker/canvas-apps/connections/connection-sharepoint-online), and set set its mode to New:

![](/post/2020-06-16-controlling-data-access-leveraving-powerbi-rls-with-powerapps.en_files/figure4.png)

The form includes all the fields in our SharePoint list by default; however, if this does not happen, we have to select the following fields:

- Title
- Project
- User
- E-mail
- user_text
- Date

![](/post/2020-06-16-controlling-data-access-leveraving-powerbi-rls-with-powerapps.en_files/figure6.png)

The interface I set up can be seen below, where only the Project and the User fields appear for the user to be able to insert. The rest of the fields are hidden and their values are being computed automatically.

![](/post/2020-06-16-controlling-data-access-leveraving-powerbi-rls-with-powerapps.en_files/figure7.png)

On the navigation pane we can see all the fields that are present in the PowerAPP (note that there might be additional fields within this app). The main fields mentioned before which are hidden have default values so they derive what information they are recording from the answers given.


![](/post/2020-06-16-controlling-data-access-leveraving-powerbi-rls-with-powerapps.en_files/figure12.png)

- Title: This field serves as an identificator field which stores information about the project, the person, and the date it was recorded. The value is derived from fields inside the PowerApp.

![](/post/2020-06-16-controlling-data-access-leveraving-powerbi-rls-with-powerapps.en_files/figure9.png)

- Project: this field was set up so it gets its value from a dropdown list created within the App. The dropdown values can be populated from [another document](http://blog.extrobe.co.uk/blog/2016/11/27/populate-a-dropdown-based-on-the-value-of-another-dropdown-in-powerapps/) or they [can be inserted manually](https://www.powerobjects.com/blog/2017/03/06/how-to-create-an-option-set-in-powerapps/).

![](/post/2020-06-16-controlling-data-access-leveraving-powerbi-rls-with-powerapps.en_files/figure10.png)

- User: We do not modify this field to allow it to search for users within the firm.

- E-mail: We retrieve the e-mail by referencing the user field and pulling its e-mail.

![](/post/2020-06-16-controlling-data-access-leveraving-powerbi-rls-with-powerapps.en_files/figure14.png)

- Date access: In this case,  I created the date acess so I would now when the permissions are being given, and set-it up so it records today's date.

![](/post/2020-06-16-controlling-data-access-leveraving-powerbi-rls-with-powerapps.en_files/figure15.png)

- user_text: retrieves the name of the user in text form, referencing the user field.

![](/post/2020-06-16-controlling-data-access-leveraving-powerbi-rls-with-powerapps.en_files/figure13.png)

Finally, we add a button at the end which will **submit and reset the form**.

#### 1.2.2. PowerApp elements: Gallery


We add a **gallery** which will display people who are present in the projects. First, we insert a gallery and link it to our data.

![](/post/2020-06-16-controlling-data-access-leveraving-powerbi-rls-with-powerapps.en_files/figure5.png)

The interface I created contains:

- Picture
- Name
- Job title
- Remove button

![](/post/2020-06-16-controlling-data-access-leveraving-powerbi-rls-with-powerapps.en_files/figure18.png)
![](/post/2020-06-16-controlling-data-access-leveraving-powerbi-rls-with-powerapps.en_files/figure19.png)

The functionalities behind each field is mostly built within the Gallery and I will not go in deep. However, the remove button serves to remove records from the list, by using the following syntax:

![](/post/2020-06-16-controlling-data-access-leveraving-powerbi-rls-with-powerapps.en_files/figure20.png)

Finally, this gallery has to be filtered according to the selected project, which is done by selecting the following items inside its data properties:

![](/post/2020-06-16-controlling-data-access-leveraving-powerbi-rls-with-powerapps.en_files/figure21.png)

The function sorts by user name, selecting only rows present in the selected project. It must be noted that there is a limit on SharePoint lists on the number of rows, and the rule of thumb is that if you want to have over 2,000 records you should migrate where the data is being stored.

After finishing this App, don't forget to share it with the people whom will be in charge of giving/removing access (and giving them access to modify the SharePoint list as well).

### 2. PowerBI integration

Based on [this article](https://radacad.com/dynamic-row-level-security-with-profiles-and-users-in-power-bi), we set-up a dynamic row-level-security for the report with DAX. The idea behind this is to have a both **users** and **profiles** from which the filters are being applied to the data.

Our first step is to [import the SharePoint list into PowerBI](https://docs.microsoft.com/en-us/power-bi/connect-data/desktop-sharepoint-online-list).

We then set up all the permissions in our App, and prepare to connect it to our PowerBI report. After we import the data we set up two tables: **user_project** which has a record of all the users with their e-mails. I applied a quick R-script to have a clean table but this is not necessary:

![](/post/2020-06-16-controlling-data-access-leveraving-powerbi-rls-with-powerapps.en_files/figure23.png)

And a table which contains **unique_users**. This data does need some manipulation, as we need to have only unique users. I applied an R-script here as well, but this can be easily done with DAX.

![](/post/2020-06-16-controlling-data-access-leveraving-powerbi-rls-with-powerapps.en_files/figure24.png)

We connect this tables with their meaningful relationship. This means, the project filter must be set-up correctly so it propagates to the data-model. Refer to the article on RLS for more information, the data model should have the following structure (assuming star-schema).

![](/post/2020-06-16-controlling-data-access-leveraving-powerbi-rls-with-powerapps.en_files/figure25.png)

Finally, we set up the RLS permissions by setting up roles:

![](/post/2020-06-16-controlling-data-access-leveraving-powerbi-rls-with-powerapps.en_files/figure27.png)

### 3. Sharing the report

After publishing the report, we then have to share it in the PowerBI online interface.
In order to properly share this report we have to:

1. Share the report itself

![](/post/2020-06-16-controlling-data-access-leveraving-powerbi-rls-with-powerapps.en_files/figure28.png)

2. Set-up RLS within the dataset: Go to datasets/Security, and then assign the corresponding RLS to the users:

![](/post/2020-06-16-controlling-data-access-leveraving-powerbi-rls-with-powerapps.en_files/figure29.png)

![](/post/2020-06-16-controlling-data-access-leveraving-powerbi-rls-with-powerapps.en_files/figure30.png)

3. Share the datasest

![](/post/2020-06-16-controlling-data-access-leveraving-powerbi-rls-with-powerapps.en_files/figure31.png)

This article integrates many Microsoft elements: PowerApps, PowerBI, SharePoint, in order to obtain an easily managed RLS control for PowerBI reports. The explanations were performed at a very high-level and a basic/intermediate knowledge of these tools is required.

It successfully creates a tool that can be used by anyone within the organization, minimizing human error and automating data update processes.





















