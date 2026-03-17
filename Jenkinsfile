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
    docker.image('instrumentisto/rsync-ssh').inside('-u root') {
        sshagent (credentials: ['ssh-prod']) {
            sh '''
            set -e

            mkdir -p ~/.ssh
            chmod 700 ~/.ssh

            ssh-keyscan -H prod.kelasdevops.xyz >> ~/.ssh/known_hosts 2>/dev/null || true

            echo "=== TEST SSH ==="
            ssh -o StrictHostKeyChecking=no ajiiee@prod.kelasdevops.xyz "echo CONNECTED"

            echo "=== MULAI RSYNC ==="
            rsync -rav --delete \
            -e "ssh -o StrictHostKeyChecking=no" \
            ./ ajiiee@prod.kelasdevops.xyz:/home/ajiiee/deploy/ \
            --exclude=.env \
            --exclude=storage \
            --exclude=.git \
            --exclude=node_modules \
            --exclude=vendor

            echo "=== RSYNC SELESAI ==="

            ssh -o StrictHostKeyChecking=no ajiiee@prod.kelasdevops.xyz << 'EOF'
                cd /home/ajiiee/deploy
                composer install --no-dev --optimize-autoloader
                php artisan config:cache
            EOF
            '''
        }
    }
}
}