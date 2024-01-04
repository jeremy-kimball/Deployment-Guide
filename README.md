# Deployment Guide with Railway
Deploy a .NET 6 Web app with a Postgres database to Railway
## ❗Disclaimer❗
I've really just finished getting everything working, so this is the solution without any revisions. Probably lots of areas for improvements and less than elegant solutions, but hey it works.
## Prerequisites
1. Railway account
2. Project is on Github

# Lets Get Started
  ### 1. After creating and logging into a Railway Account, from your dashboard select new project <br>
  ![NewProject](https://github.com/jeremy-kimball/Deployment-Guide/assets/130601077/82295dd3-4b73-423b-8cbc-e3ba6a4553ee)
  ### 2. Select Deploy from Github Repo <br>
  ![image](https://github.com/jeremy-kimball/Deployment-Guide/assets/130601077/cee8fdb1-a48d-481a-8c9a-a7a6639907d9)
  ### 3. Select the Repo containing your application <br>
  ![SelectRepo](https://github.com/jeremy-kimball/Deployment-Guide/assets/130601077/8ef23563-7bb5-431b-b9d2-5f217661ffd3)
  ### 4. Inside the new project right click the background and select Database under add new service <br>
  ![image](https://github.com/jeremy-kimball/Deployment-Guide/assets/130601077/220fd924-764f-4369-a523-af87382e3496)
  ### 5. Select Postgres
  ![image](https://github.com/jeremy-kimball/Deployment-Guide/assets/130601077/5c67883a-d203-476c-8822-0b81eb041445)
### 6. Select the Postgres block and select the variables tab
![image](https://github.com/jeremy-kimball/Deployment-Guide/assets/130601077/9a1ecbf9-4bc8-41fa-9f10-a797933f489c)
### 7. Copy the following variables, PGHOST, DATABASE_URL, PGPORT, POSTGRES_USER, PGPASSWORD
## 8. Select the block containing your application and add these variables with the same names and values as in the postgres block. Add an additional variable here with the key PORT and value 3000
![image](https://github.com/jeremy-kimball/Deployment-Guide/assets/130601077/08d8433c-9df6-44d3-b870-d9cbc940f30b)
# Application
1. Create a new class named ConnectionHelper. This will check the environment variables that you set inside railway. If those values are null it can be assumed the application is launching locally and will instead use the local connection string. <br>
2.❗IMPORTANT❗ Make sure to change the database value in the local connection string to match the name of your database, optionally change the MYCONNECTIONSTRING to your desired name.
```
public static class ConnectionHelper
{
    public static string getConnectionString()
    {
        string MYCONNECTIONSTRING = "";
        if (Environment.GetEnvironmentVariable("PGHOST") != null)
        {
            MYCONNECTIONSTRING = 
                $"Server={Environment.GetEnvironmentVariable("PGHOST")};" +
                $"Database={Environment.GetEnvironmentVariable("DATABASE_URL")};" +
                $"Port={Environment.GetEnvironmentVariable("PGPORT")};" +
                $"Username={Environment.GetEnvironmentVariable("POSTGRES_USER")};" +
                $"Password={Environment.GetEnvironmentVariable("PGPASSWORD")}";
        }
        else
        {
            MYCONNECTIONSTRING = "Server=localhost;Database=MYDATABASE;Port=5432;Username=postgres;Password=password123";
        }
        return MYCONNECTIONSTRING;
    }
}
```
3. Alter the following line in program.cs to match the following, changing MYDBCONTEXT and MYDBNOTFOUND to match your existing names
```
builder.Services.AddDbContext<MYDBCONTEXT>(
    options =>
        options
            .UseNpgsql(ConnectionHelper.getConnectionString()
                    ?? throw new InvalidOperationException(
                            "Connection String 'MYDBNOTFOUND' not found"
                            )
                    )
                    .UseSnakeCaseNamingConvention()
                    );
```
4. Below the line `var app = builder.Build();` in program.cs
Add the following block of code, this is the programmatic version of `update-database` and so ensures our remote DB has our migrations. Again changing MYDBCONTEXT to match your existing one.
```
using (var scope = app.Services.CreateScope())
{
    var dbContext = scope.ServiceProvider.GetRequiredService<MYDBCONTEXT>();
    // This will throw an exception if the connection fails
    dbContext.Database.Migrate();
}
```
# Connecting to the Remote DB in pgAdmin
1. Right click servers, go to register, and finally server...
![image](https://github.com/jeremy-kimball/Deployment-Guide/assets/130601077/b426cf1d-0a26-4a3d-a7c4-c2284b39328d)
2. Inside this window give the server a name (doesn't matter what it is)
![image](https://github.com/jeremy-kimball/Deployment-Guide/assets/130601077/7768c19c-52fa-49e5-98dd-33098b0b1c4e)
3. Head to the connection tab
Here we will be using values from the variables tab from Railway
- Host name/address = PGHOST
- PORT = PGPORT
- Username = PGUSERNAME
- Password = PGPASSWORD
After this, hit save.
![image](https://github.com/jeremy-kimball/Deployment-Guide/assets/130601077/b6b3abe4-5c52-44cd-8021-e7043458d0c8)
4. This is the connection you are looking for <br>
![image](https://github.com/jeremy-kimball/Deployment-Guide/assets/130601077/ea9be6be-8a44-4825-8e47-05038d978e6e)
<br></br>
5. You are finished and can now write your queries in here!
# Common Issues
1. CSS not showing <br/>
Make this adjustment in your program.cs file, on your builder.
```
var builder = WebApplication.CreateBuilder(new WebApplicationOptions()
{
    Args = args,
    ContentRootPath = "/app/out",
    WebRootPath = "wwwroot",
});
```
