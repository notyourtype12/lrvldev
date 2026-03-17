node {
    checkout scm

    stage("Build") {
        docker.image('composer:2').inside('-u root') {
            sh '''
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
        docker.image('composer:2').inside('-u root') {
            sh '''
            php -v
            composer -v
            '''
        }
    }

    stage('Deploy') {
        docker.image('instrumentisto/rsync-ssh').inside('-u root') {
            sshagent (credentials: ['ssh-prod']) {
                sh '''
                # Setup SSH
                mkdir -p ~/.ssh
                chmod 700 ~/.ssh
                ssh-keyscan -H localhost >> ~/.ssh/known_hosts

                # Deploy ke localhost
                rsync -rav --delete \
                -e "ssh -o StrictHostKeyChecking=no" \
                ./ ajiiee@localhost:/home/ajiiee/deploy/ \
                --exclude=.env \
                --exclude=storage \
                --exclude=.git \
                --exclude=node_modules \
                --exclude=vendor

                ssh ajiiee@localhost << 'EOF'
                    cd /home/ajiiee/deploy
                    composer install --no-dev --optimize-autoloader
                    php artisan config:cache
                    php artisan route:cache
                    php artisan view:cache
                EOF
                '''
            }
        }
    }
}