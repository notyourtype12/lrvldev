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
            set -e

            mkdir -p ~/.ssh
            chmod 700 ~/.ssh

            # FIX: redirect + anti error
            ssh-keyscan -H 172.24.153.187 >> ~/.ssh/known_hosts 2>/dev/null

            echo "=== TEST SSH ==="
            ssh -o StrictHostKeyChecking=no ajiiee@172.24.153.187 "echo CONNECTED"

            echo "=== MULAI RSYNC ==="
            rsync -rav --delete \
            -e "ssh -o StrictHostKeyChecking=no" \
            ./ ajiiee@172.24.153.187:/home/ajiiee/deploy/ \
            --exclude=.env \
            --exclude=storage \
            --exclude=.git \
            --exclude=node_modules \
            --exclude=vendor

            echo "=== RSYNC SELESAI ==="

            ssh ajiiee@172.24.153.187 << 'EOF'
                cd /home/ajiiee/deploy
                composer install --no-dev --optimize-autoloader
                php artisan config:cache
            EOF
            '''
        }
    }
}
}