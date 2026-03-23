pipeline {
    agent any
    // VARIABLES DE ENTORNO
    environment {
        APP_NAME       = 'jenkins-demo'
        VERCEL_TOKEN = credentials('VERCEL_TOKEN')
    }

    // OPCIONES GENERALES
    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timestamps()
        timeout(time: 15, unit: 'MINUTES')
        disableConcurrentBuilds()
    }

    // ETAPAS
    stages {

        // CHECKOUT
        stage('Checkout') {
            steps {
                echo "📥 Clonando repositorio — rama: ${env.GIT_BRANCH} | commit: ${env.GIT_COMMIT.take(7)}"
                checkout scm
            }
        }

        // LINT
        stage('Lint') {
            steps {
                echo 'Analizando HTML y CSS'
                sh '''
                    # ── HTML ──
                    echo "=== Lint HTML ==="
                    HTML_ERRORS=0
                    for file in *.html; do
                        if ! grep -qi "<!DOCTYPE html>" "$file"; then
                            echo "  ⚠️  Falta DOCTYPE en: $file"
                            HTML_ERRORS=$((HTML_ERRORS + 1))
                        fi
                        if ! grep -qi "<title>" "$file"; then
                            echo "  ⚠️  Falta <title> en: $file"
                            HTML_ERRORS=$((HTML_ERRORS + 1))
                        fi
                        echo "  ✅ Revisado: $file"
                    done
                    if [ $HTML_ERRORS -gt 0 ]; then
                        echo "❌ Se encontraron $HTML_ERRORS problema(s) en HTML."
                        exit 1
                    fi

                    # ── CSS ──
                    echo "=== Lint CSS ==="
                    for file in css/*.css; do
                        if [ -s "$file" ]; then
                            echo "  ✅ CSS no vacío: $file"
                        else
                            echo "  ⚠️  Archivo CSS vacío: $file"
                        fi
                    done

                    echo "✅ Lint completado sin errores"
                '''
            }
        }

        // DEPLOY

        stage('Deploy') {
            tools {
                nodejs 'NodeJS'
            }
            steps {
                echo "🚀 Desplegando a Vercel..."
                sh '''
                    npm i -g vercel
                    vercel --prod --token $VERCEL_TOKEN --yes
                '''
            }
        }
    }

    // POST
    post {
        always {
            cleanWs()
        }

        success {
            echo "✅ Build #${env.BUILD_NUMBER} exitoso"
        }

        failure {
            echo "❌ Build #${env.BUILD_NUMBER} fallido"
        }
    }

}
