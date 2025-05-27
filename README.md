# Spring Boot - Thymeleaf & TailwindCSS

I wanted to use Thymeleaf because I wanted to a server-side rendered web application.

While I'm familiar with using a frontend framework like React. I still want to explore
a more traditional server rendered approach just because.

I wanted to see how far I'll go with it and see how it works.

## How I set up Thymeleaf + Tailwind

1. Create an application via [Spring Initializr](https://start.spring.io/)
2. Visit the [Tailwind Get Started Page](https://v3.tailwindcss.com/docs/installation) and follow the instruction there
   but there will be some modifications.

**_Note:_** The important part here is the `content`. We're telling Tailwind to look at the `resources/templates`
directory for
html files.

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: ["./src/main/resources/templates/**/*.{html,js}"],
  theme: {
    extend: {},
  },
  plugins: [],
};
```

3. Create the `tailwind.css` file containing the following:

**_Note:_** I am using Tailwind v3 in this repository. Update when necessary.

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

4. Add the following scripts in your `package.json`

**_Note:_** Tailwind will use the `tailwind.css` file and place the output css in the the `resources/static/css` for the
views to use.

```json
{
  "scripts": {
    "build": "npx tailwindcss -i ./tailwind.css -o ./src/main/resources/static/css/main.css"
  }
}
```

5. Run `npm run build` to test out Tailwind and there you have it!

## Integrate `npm run build` in Gradle Build

1. Add the `com.github.node-gradle.node` plugin in `build.gradle`

**_Note:_** The version of the plugin will vary.

```groovy
id 'com.github.node-gradle.node' version '7.1.0'
```

2. Add the following tasks to `build.gadle`

**_Note:_** You're including `npm run build` so that it would compile and generate the `main.css` file that the views
are going to use for styling.

```groovy
node {
    download.set(true)
    version.set('22.16.0')  // Pick the LTS version
}

tasks.register('npmRunBuild', NpmTask) {
    args = ['run', 'build']
    dependsOn npmInstall
}

tasks.named('processResources') {
    dependsOn tasks.npmRunBuild
}
```

3. Run `.\gradle build` to test if you've successfully integrated `npmRunBuild` as part of the build process.

## Resolve View Caching! Stop Restarting the Server

When you make updates to your views, most of the time, you have to restart the server to reflect changes.

**You don't have to do that!** The reason why it doesn't reflect your changes is because the first render is cached.

To resolve this you must turn of caching!

1. Create a configuration for your local development.
   **_Note:_** I got this idea from [bootify.io](https://bootify.io/)! It's pure genius.

```java

@Configuration
@Profile("local")
public class LocalDevConfig {

    public LocalDevConfig(final TemplateEngine templateEngine) throws IOException {
        final ClassPathResource applicationYml = new ClassPathResource("application.yml");
        if (applicationYml.isFile()) {
            File sourceRoot = applicationYml.getFile().getParentFile();
            while (sourceRoot.listFiles((dir, name) -> name.equals("gradlew")).length != 1) {
                sourceRoot = sourceRoot.getParentFile();
            }
            final FileTemplateResolver fileTemplateResolver = new FileTemplateResolver();
            fileTemplateResolver.setPrefix(sourceRoot.getPath() + "/src/main/resources/templates/");
            fileTemplateResolver.setSuffix(".html");
            fileTemplateResolver.setCacheable(false);
            fileTemplateResolver.setCharacterEncoding("UTF-8");
            fileTemplateResolver.setCheckExistence(true);
            templateEngine.setTemplateResolver(fileTemplateResolver);
        }
    }

}
```

2. Add `spring.profiles.active=local` to your environment variable when you run the appliation to apply the
   configuration.

**With the configuration, you're code changes in the html files will be reflected with a simple refresh of the page!**
You don't need to restart the server to reflect the changes in the html files anymore. 