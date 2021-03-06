# CommunicationLib

This package has been built to reduce boilerplate communication work, it also incorporates interface versioning(i recently update the package to .net core 6 but haven't tested it yet).

This package currently has support for:
 - HTTP communication.
 - Websocket communication(Wamp).

i might add support for the following protocols:
 - RabbitMQ
 - MQTT

to use the package for wamp add the following to startup file:

```
 public class Startup
    {
        private readonly IConfigurationRoot configuration;
        private readonly MetadataCollection metadataCollection;
        private const int MinVersion = 1;
        private const int MaxVersion = 2;

        public Startup(IHostingEnvironment env)
        {
            ...
            
            metadataCollection = new MetadataCollection(
                new[] { typeof(ILoginService) },
                "com.pca",
                MinVersion, MaxVersion
            );
            
            ...
        }

        public void ConfigureServices(IServiceCollection services)
        {
            ...
            
            
            
            services.Configure<WampSettings>(configuration.GetSection("Wamp"));
            
            services.AddMetadataCollection(metadataCollection)
                    .AddWamp()
                    .AddTransient<ILoginService, LoginService>();
                    
                    
                    
            ...

        }

        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, ILoggerFactory loggerFactory, IServiceProvider serviceProvider)
        {
            ...
            
            app.UseMiddleware<MetadataEndpointMiddleware>();
            app.UseWamp();
            app.UseHttp();

            ...
        }
```        
        
        
# Interface versioning

Add "[ServiceVersion(x)]" to your interface like this
```
    public interface ILoginService
    {
        [ServiceVersion(1)]
        Task<String> Login(string email, string password);
        
        [ServiceVersion(2)]
        Task<String> Login(string email, string password, string 2fa);
    }
```

When u made a new version of your interface and u increased the ServiceVersion number to for example 3 do not forget to update the MaxVersion in the startup file. This also works the other way around increase MinVersion to 2 so the ServiceVersion 1 of you're interface won't be used.

# Metadata endpoint

If u browse to http://localhost:22224/metadata u will see a page with a json object which contains all Interfaces/services with parameters etc(be sure to add the interface to the metadatacollection see startup.cs).

Based on the metadata we can create a simple console application which reads the json and generates all typescript files/models to easily update the frontend for currently supported protocols.
