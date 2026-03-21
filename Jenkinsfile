pipeline {
    agent any
    // VARIABLES DE ENTORNO
    environment {
        APP_NAME       = 'jenkins-demo'
        DOCKER_IMAGE   = "jenkins-demo:${env.BUILD_NUMBER}"
        CONTAINER_PORT = '8080'   // Puerto expuesto en el host
        NGINX_PORT     = '80'     // Puerto interno de Nginx
        VERCEL_TOKEN = credentials('VERCEL_TOKEN')
        // Email
        NOTIFY_EMAIL = 'agarridog1@miumg.edu.gt'   // <-- cambia esto
    }

    // TRIGGERS
    triggers {
        githubPush()   // Requiere plugin "GitHub" + webhook en el repo
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

        // ── 1. CHECKOUT ────────────────────────────
        stage('Checkout') {
            steps {
                echo "📥 Clonando repositorio — rama: ${env.GIT_BRANCH} | commit: ${env.GIT_COMMIT.take(7)}"
                checkout scm
            }
        }

        // ── 2. LINT ────────────────────────────────
        stage('Lint') {
            steps {
                echo '🔍 Analizando HTML, CSS y JS...'
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

                    echo "✅ Lint completado sin errores críticos"
                '''
            }
        }

        // ── 5. DEPLOY ──────────────────────────────

        stage('Deploy') {
            tools {
                nodejs 'NodeJS'
            }
            steps {
                sh '''
                    npm i -g vercel
                    vercel --prod --token $VERCEL_TOKEN --yes
                '''
            }
        }
    }


    // ──────────────────────────────────────────────
    // POST — Notificaciones y limpieza
    // ──────────────────────────────────────────────
    post {
        always {
            sh 'rm -rf dist/ 2>/dev/null || true'
            cleanWs()
        }

        success {
            echo "✅ Build #${env.BUILD_NUMBER} exitoso"

            // Notificación Email (plugin: Email Extension)
            emailext(
                subject: "✅ [Jenkins] ${env.APP_NAME} — Build #${env.BUILD_NUMBER} EXITOSO",
                body: """
                    <h2 style="color:#28a745">✅ Deploy Exitoso</h2>
                    <table>
                        <tr><td><b>Proyecto:</b></td><td>${env.APP_NAME}</td></tr>
                        <tr><td><b>Build:</b></td><td>#${env.BUILD_NUMBER}</td></tr>
                        <tr><td><b>Rama:</b></td><td>${env.GIT_BRANCH}</td></tr>
                        <tr><td><b>Commit:</b></td><td>${env.GIT_COMMIT.take(7)}</td></tr>
                    </table>
                    <br/><a href="${env.BUILD_URL}">Ver detalles del build →</a>
                """,
                mimeType: 'text/html',
                to: env.NOTIFY_EMAIL
            )
        }

        failure {
            echo "❌ Build #${env.BUILD_NUMBER} fallido"

            // Limpiar contenedor de test si quedó colgado
            sh "docker stop ${env.APP_NAME}-test 2>/dev/null || true"
            sh "docker rm   ${env.APP_NAME}-test 2>/dev/null || true"

            // Notificación Email
            emailext(
                subject: "❌ [Jenkins] ${env.APP_NAME} — Build #${env.BUILD_NUMBER} FALLIDO",
                body: """
                    <h2 style="color:#dc3545">❌ Build Fallido</h2>
                    <table>
                        <tr><td><b>Proyecto:</b></td><td>${env.APP_NAME}</td></tr>
                        <tr><td><b>Build:</b></td><td>#${env.BUILD_NUMBER}</td></tr>
                        <tr><td><b>Rama:</b></td><td>${env.GIT_BRANCH}</td></tr>
                        <tr><td><b>Commit:</b></td><td>${env.GIT_COMMIT.take(7)}</td></tr>
                    </table>
                    <br/><a href="${env.BUILD_URL}console">Ver logs del error →</a>
                """,
                mimeType: 'text/html',
                to: env.NOTIFY_EMAIL
            )
        }
    }

}
