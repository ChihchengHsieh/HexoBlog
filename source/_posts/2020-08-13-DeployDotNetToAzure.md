---
title: Deploy .NET core to Azue
tags:
    - .NET
categories: App Development 
photos:
    - https://www.interlinked.com.au/wp-content/uploads/2016/12/Microsoft-Azure-Interlinked-2.png
---


# Open the release setting window


## Find the release window under the **Build** tab:

![](https://trello.com/1/cards/5f33bb853323af6567cb8bd2/attachments/5f341a07d8f8cd2d1183bb6c/previews/5f341a07d8f8cd2d1183bb7c/download?signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJuYmYiOjE1OTcyNDgwMDAsImV4cCI6MTU5NzI1MzQwMCwicmVzIjoiNWYzM2JiODUzMzIzYWY2NTY3Y2I4YmQyOjVmMzQxYTA3ZDhmOGNkMmQxMTgzYmI2Yzo1ZjM0MWEwN2Q4ZjhjZDJkMTE4M2JiN2MiLCJhdWQiOiJUcmVsbG8iLCJpc3MiOiJUcmVsbG8ifQ.t_Yc-PI3wl70YyBVgWxv0BPSM06jDHHyn7HRSY9A_4c)

## Pressing the start button to finish the publishing process.

![](https://trello.com/1/cards/5f33bb853323af6567cb8bd2/attachments/5f341ab083e2162d80ff96df/previews/5f341ab183e2162d80ff96ea/download?signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJuYmYiOjE1OTcyNDgwMDAsImV4cCI6MTU5NzI1MzQwMCwicmVzIjoiNWYzM2JiODUzMzIzYWY2NTY3Y2I4YmQyOjVmMzQxYWIwODNlMjE2MmQ4MGZmOTZkZjo1ZjM0MWFiMTgzZTIxNjJkODBmZjk2ZWEiLCJhdWQiOiJUcmVsbG8iLCJpc3MiOiJUcmVsbG8ifQ.kazktpZR_jCx5uYRI20XFgZjgM0Z3J7MiF95Nvv4LC4)

After pressing this start button, a window will pop out. And, just follow the instruction to fill and the fields.  

## Set up the dependencies
After you complete the fields about your sever. It will point out the dependency you need. You can press **Configure** to set up your dependency. And you are good to go. Just press the Publish button you can publish your serve to Azure cloud.

![](https://trello.com/1/cards/5f33bb853323af6567cb8bd2/attachments/5f341be22bb9dd602e23ec6e/previews/5f341be32bb9dd602e23ec96/download?signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJuYmYiOjE1OTcyNDgwMDAsImV4cCI6MTU5NzI1MzQwMCwicmVzIjoiNWYzM2JiODUzMzIzYWY2NTY3Y2I4YmQyOjVmMzQxYmUyMmJiOWRkNjAyZTIzZWM2ZTo1ZjM0MWJlMzJiYjlkZDYwMmUyM2VjOTYiLCJhdWQiOiJUcmVsbG8iLCJpc3MiOiJUcmVsbG8ifQ.o33-yzoVFGc2Xuk4Lc6xWdqRgSjAaMZanAvzpp2PyZI)

# Bugs

## HTTP Error 500.30 - ANCM In-Process Start Failure


### Solution
This may happen when publishing the server to Azure. This message means the server on the azure **doesn't start correctly**. Therefore, any request to the server will get this message as response. To understand what's going on on the Azure server, we have to first go to the **Azure console** and find the console to run the solution again here. 

![](https://trello.com/1/cards/5f33bb853323af6567cb8bd2/attachments/5f3417dd6ec31158f0667f73/previews/5f3417de6ec31158f0667f8e/download?signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJuYmYiOjE1OTcyNDgwMDAsImV4cCI6MTU5NzI1MzQwMCwicmVzIjoiNWYzM2JiODUzMzIzYWY2NTY3Y2I4YmQyOjVmMzQxN2RkNmVjMzExNThmMDY2N2Y3Mzo1ZjM0MTdkZTZlYzMxMTU4ZjA2NjdmOGUiLCJhdWQiOiJUcmVsbG8iLCJpc3MiOiJUcmVsbG8ifQ.6gpvEbQ8VCKrBcB-_Ot19uncgdRpRtbSCCSH93-5bSM)

To run the solution again in this console, we can get the same result of the output window during debugging locally. According to the demonstrated message above, the bug is prababaly caused by the **Mirgation Error**. Since the data in the database is jus for the use of testing, we can directly remove all the migrations and migrate them when the app start by the following code in `Program.cs`. 

```csharp
  public static void Main(string[] args)
        {
            Console.WriteLine("Running Start");
            IHost host = CreateHostBuilder(args).Build();

            // Intialise the SQL Db before running the Host
            using (IServiceScope scope = host.Services.CreateScope())
            {
                try
                {
                    DivingAPIContext context = scope.ServiceProvider.GetService<DivingAPIContext>();

                    // Ensuring the Db Status
                    context.Database.EnsureCreated();
                    context.Database.Migrate();

                }
                catch (Exception ex)
                {
                    ILogger<Program> logger = scope.ServiceProvider.GetRequiredService<ILogger<Program>>();
                    logger.LogError(ex, "An error occured while Operating the Db");
                    throw;
                }
            }

            host.Run();
        }
```


### Steps:

- Remove the migrations and database. (If you don't want to remove the database, you can update your database to previous version and do the same tricks below).
- Run migration during the app start. 

### Another solution (Not working on my case, but may help your):
[[Click here to see another solution]](https://build5nines.com/fix-net-core-http-error-500-30-after-publish-to-app-service-from-visual-studio/)


