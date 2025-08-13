import os
import re
import sys

def validate_keys(token, project_key):
    if not token.startswith("sonar_") or len(token) < 30:
        print("❌ Token de SonarCloud inválido.")
        return False
    if len(project_key) < 5 or " " in project_key:
        print("❌ Project Key inválido.")
        return False
    return True

def create_workflows(token, project_key):
    os.makedirs(".github/workflows", exist_ok=True)

    ci_content = f"""name: Android CI
on: [push, pull_request]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Set up JDK
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'
      - name: Build with Gradle
        run: ./gradlew build
"""
    with open(".github/workflows/android-ci.yml", "w") as f:
        f.write(ci_content)

    sonar_content = f"""name: SonarCloud
on:
  push:
    branches: [ main ]
jobs:
  sonarcloud:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: SonarCloud Scan
        uses: SonarSource/sonarcloud-github-action@master
        with:
          args: >
            -Dsonar.projectKey={project_key}
            -Dsonar.organization=tu_org
        env:
          SONAR_TOKEN: {token}
"""
    with open(".github/workflows/sonarcloud.yml", "w") as f:
        f.write(sonar_content)

    print("✅ Workflows creados en .github/workflows/")

def main():
    token = input("🔑 Ingresa tu SonarCloud Token: ").strip()
    project_key = input("📛 Ingresa tu SonarCloud Project Key: ").strip()

    if not validate_keys(token, project_key):
        sys.exit(1)

    create_workflows(token, project_key)

    if not os.path.exists("README.md"):
        with open("README.md", "w") as f:
            f.write("# JarvisApp\nAsistente inteligente para Android.\n")

    if not os.path.exists(".gitignore"):
        with open(".gitignore", "w") as f:
            f.write("*.apk\n/build/\n/.idea/\n")

if __name__ == "__main__":
    main()

