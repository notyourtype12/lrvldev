node {
    checkout scm

    stage("Build") {
        docker.image('composer:2').inside('--entrypoint="" -u root') {
            sh '''
            git config --global --add safe.directory /var/lib/jenkins/workspace/lrvll

            composer install --no-interaction --prefer-dist --optimize-autoloader
            '''
        }
    }

    stage("Test") {
        docker.image('ubuntu:22.04').inside('-u root') {
            sh 'echo "Ini adalah test"'
        }
    }

    stage("PHP Check") {
        docker.image('composer:2').inside('--entrypoint="" -u root') {
            sh '''
            php -v
            composer -v
            '''
        }
    }

   stage('Deploy') {
    sh '''
    echo "=== DEPLOY LOCAL ==="

    mkdir -p /var/lib/jenkins/deploy

    rsync -rav --delete \
    ./ /var/lib/jenkins/deploy/ \
    --exclude=.env \
    --exclude=storage \
    --exclude=.git \
    --exclude=node_modules \
    --exclude=vendor

    cd /var/lib/jenkins/deploy

    composer install --no-dev --optimize-autoloader

    # buat folder yang dibutuhkan laravel
    mkdir -p storage/framework/views
    mkdir -p storage/framework/cache
    mkdir -p storage/framework/sessions
    mkdir -p storage/logs

    chmod -R 775 storage bootstrap/cache

    php artisan config:cache
    php artisan route:cache
    php artisan view:cache
    php artisan migrate --force
    '''
}
}