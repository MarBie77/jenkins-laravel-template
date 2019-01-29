# jenkins-laravel-template

Template build configuration files for use with the docker-jenkins-php project

- build.xml: Apache Ant build file, see <https://ant.apache.org/manual/using.html>
- build/phpcs.xml: config file for PHP Code Sniffer <https://github.com/squizlabs/PHP_CodeSniffer>
- build/phpdox.xml: config file for PHP Documentation Generator <http://phpdox.net/>
- build/phpmd.xml: config file for PHP Mess Detector <https://phpmd.org/>

## Usage

add build-directory and build.xml to your GIT-project.

> Do NOT forget the .gitignore file in the build-directory, otherwise you'll have all reports in your next commit.

### optional steps

1. change the name of the project php-template to your real project name
2. change the .env-filename in the target "env-production" (or just remove target from build lists)