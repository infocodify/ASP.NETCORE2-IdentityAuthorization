# ASP.NETCORE2-IdentityAuthorization
Identity Authorization in asp.Net Core 2.

### Create Autentication and Authorization in asp.Net Core 2 follow the steps:

1.  Create .Net Core 2 web application project with Individual User Account Authentication

2.  Add the folllowing code to the Startup.cs file

```csharp
    public class Startup
    {
        public Startup(IConfiguration configuration)
        {
            Configuration = configuration;
        }

        public IConfiguration Configuration { get; }

        // This method gets called by the runtime. Use this method to add services to the container.
        public void ConfigureServices(IServiceCollection services)
        {
          //use in memory database for testing purpose
            services.AddDbContext<ApplicationDbContext>(options =>
               options.UseInMemoryDatabase("AuthSample"));

            services.AddIdentity<ApplicationUser, IdentityRole>()
                .AddEntityFrameworkStores<ApplicationDbContext>()
                .AddDefaultTokenProviders();

            services.AddMvc()
                .AddRazorPagesOptions(options =>
                {
                    options.Conventions.AuthorizeFolder("/Account/Manage");
                    options.Conventions.AuthorizePage("/Account/Logout");
                });

            // Register no-op EmailSender used by account confirmation and password reset during development
            // For more information on how to enable account confirmation and password reset please visit https://go.microsoft.com/fwlink/?LinkID=532713
            services.AddSingleton<IEmailSender, EmailSender>();
        }

        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
                app.UseBrowserLink();
                app.UseDatabaseErrorPage();
            }
            else
            {
                app.UseExceptionHandler("/Error");
            }

            app.UseStaticFiles();

            app.UseAuthentication();
            
            //initialize data stored in the SampleData class
            SampleData.InitializeData(app.ApplicationServices, loggerFactory);
          
            app.UseMvc(routes =>
            {
                routes.MapRoute(
                    name: "default",
                    template: "{controller}/{action=Index}/{id?}");
            });
        }
    }
```
3.  Create SampleData class and add the following code
```csharp
    public class SampleData
    {
        public static async Task InitializeData(IServiceProvider services, ILoggerFactory loggerFactory)
        {
            var logger = loggerFactory.CreateLogger("SampleData");

            using (var serviceScope = services.GetRequiredService<IServiceProvider>().CreateScope())
            {
                var env = serviceScope.ServiceProvider.GetService<IHostingEnvironment>();
                if (!env.IsDevelopment()) return;

                var roleManager = serviceScope.ServiceProvider.GetService<RoleManager<IdentityRole>>();

                // Create roles
                var adminTask = roleManager.CreateAsync(
                    new IdentityRole { Name = "Admin" });
                var powerUserTask = roleManager.CreateAsync(
                    new IdentityRole { Name = "Power Users" });
                Task.WaitAll(adminTask, powerUserTask);
                logger.LogInformation("==> Added Admin and Power Users roles");

                var userManager = serviceScope.ServiceProvider.GetService<UserManager<ApplicationUser>>();

                // Create default user
                var user = new ApplicationUser
                {
                    Email = "info@infocodify.com",
                    UserName = "info@infocodify.com"
                };
                await userManager.CreateAsync(user, "Infocodify2018!");
                logger.LogInformation($"==> Create user info@infocodify.com with password Infocodify2018!");

                //await userManager.AddToRoleAsync(user, "Admin");
                //await userManager.AddClaimAsync(user, new Claim(ClaimTypes.Country, "Canada"));
            }

        }

        internal static void InitializeData(IServiceProvider applicationServices, object loggerFactory)
        {
            throw new NotImplementedException();
        }
    }
```

4.  Run the application click on the Tutorials page (or your About page) and login using the default user or create a new user
