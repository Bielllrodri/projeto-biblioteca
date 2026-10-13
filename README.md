const roleOptions = [...document.querySelectorAll(".role-option")];
const identifierInput = document.querySelector("#identifier");
const identifierLabel = document.querySelector("#identifier-label");
const identifierHint = document.querySelector("#identifier-hint");
const roleDescription = document.querySelector("#role-description");
const submitButton = document.querySelector("#submit-button");
const loginForm = document.querySelector("#login-form");
const passwordInput = document.querySelector("#password");
const passwordToggle = document.querySelector(".password-toggle");
const formStatus = document.querySelector("#form-status");
const examPreviewLink = document.querySelector("#exam-preview-link");

const roleSettings = {
  aluno: {
    label: "RA do aluno",
    placeholder: "Digite seu RA",
    hint: "Informe seu Registro do Aluno.",
    description: "Área exclusiva para alunos. Use seu RA para entrar.",
    button: "Entrar como aluno",
    inputMode: "numeric",
    type: "text",
    maxLength: 30,
  },
  professor: {
    label: "CPF do professor",
    placeholder: "Digite seu CPF",
    hint: "Área exclusiva para professores. Use seu CPF para entrar.",
    description: "Área exclusiva para professores. O acesso de alunos não é permitido.",
    button: "Entrar como professor",
    inputMode: "numeric",
    type: "text",
    maxLength: 14,
  },
  gestao: {
    label: "E-mail da gestão",
    placeholder: "nome@escola.com.br",
    hint: "Área restrita à equipe de gestão escolar.",
    description: "Área restrita à gestão. O acesso de alunos e professores não é permitido.",
    button: "Entrar como gestão",
    inputMode: "email",
    type: "email",
    maxLength: 254,
  },
};

function selectRole(role) {
  const settings = roleSettings[role];
  const isTeacher = role === "professor";
  const isManagement = role === "gestao";

  roleOptions.forEach((option) => {
    const isSelected = option.dataset.role === role;
    option.classList.toggle("is-selected", isSelected);
    option.setAttribute("aria-pressed", String(isSelected));
  });

  identifierLabel.textContent = settings.label;
  identifierInput.placeholder = settings.placeholder;
  identifierInput.type = settings.type;
  identifierInput.inputMode = settings.inputMode;
  identifierInput.maxLength = settings.maxLength;
  identifierInput.value = "";
  identifierHint.textContent = settings.hint;
  roleDescription.textContent = settings.description;
  examPreviewLink.href = isTeacher ? "concursos.html" : isManagement ? "gestao.html" : "enem.html";
  examPreviewLink.textContent = isTeacher
    ? "Ir direto à preparação para concursos"
    : isManagement
      ? "Ir direto ao painel da gestão"
      : "Ir direto à preparação para o ENEM";
  submitButton.firstChild.textContent = `${settings.button} `;
  passwordInput.value = "";
  passwordInput.type = "password";
  passwordToggle.setAttribute("aria-label", "Mostrar senha");
  passwordToggle.setAttribute("aria-pressed", "false");
  formStatus.textContent = "";
  formStatus.classList.remove("is-visible");
  identifierInput.focus();
}

roleOptions.forEach((option) => {
  option.addEventListener("click", () => selectRole(option.dataset.role));
});

identifierInput.addEventListener("input", () => {
  const selectedRole = roleOptions.find((option) => option.getAttribute("aria-pressed") === "true");
  if (selectedRole?.dataset.role !== "professor") {
    return;
  }

  const digits = identifierInput.value.replace(/\D/g, "").slice(0, 11);
  identifierInput.value = digits
    .replace(/^(\d{3})(\d)/, "$1.$2")
    .replace(/^(\d{3})\.(\d{3})(\d)/, "$1.$2.$3")
    .replace(/\.(\d{3})(\d)/, ".$1-$2");
});

passwordToggle.addEventListener("click", () => {
  const shouldShowPassword = passwordInput.type === "password";
  passwordInput.type = shouldShowPassword ? "text" : "password";
  passwordToggle.setAttribute("aria-pressed", String(shouldShowPassword));
  passwordToggle.setAttribute(
    "aria-label",
    shouldShowPassword ? "Ocultar senha" : "Mostrar senha",
  );
});

loginForm.addEventListener("submit", (event) => {
  event.preventDefault();
  const selectedRole = roleOptions.find((option) => option.getAttribute("aria-pressed") === "true");
  const role = selectedRole.dataset.role;
  const identifier = identifierInput.value.trim();

  try {
    if (role === "gestao") {
      void authenticateManagement(identifier, passwordInput.value);
      return;
    }
    window.bibliotecaRegistrarAcesso({ role, identifier });
  } catch (error) {
    console.error("Não foi possível registrar o acesso neste navegador.", error);
    formStatus.textContent = "Não foi possível registrar este acesso neste navegador. Verifique as permissões do navegador e tente novamente.";
    formStatus.classList.add("is-visible");
    return;
  }

  window.location.href = "catalogo.html";
});

async function authenticateManagement(email, password) {
  submitButton.disabled = true;
  formStatus.textContent = "Verificando o acesso da gestão...";
  formStatus.classList.add("is-visible");

  try {
    const response = await fetch("/api/management/login", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      credentials: "same-origin",
      body: JSON.stringify({ email, password }),
    });
    const result = await response.json();
    if (!response.ok) {
      throw new Error(result.error || "Não foi possível autenticar a gestão.");
    }

    window.bibliotecaRegistrarAcesso({ role: "gestao", identifier: email });
    window.location.href = "gestao.html";
  } catch (error) {
    console.error("Falha na autenticação da gestão.", error);
    formStatus.textContent = error.message || "O servidor não respondeu. Verifique se ele está em execução.";
    formStatus.classList.add("is-visible");
  } finally {
    submitButton.disabled = false;
  }
}

const requestedProfile = new URLSearchParams(window.location.search).get("perfil");
if (requestedProfile && roleSettings[requestedProfile]) {
  selectRole(requestedProfile);
}
