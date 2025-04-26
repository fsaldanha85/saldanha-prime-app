import { useState } from 'react';

const checklistItems = [
  {
    category: "MANHÃ",
    items: [
      "500 ml de água ao acordar",
      "Água morna com limão + beterraba + mel + canela + pimenta",
      "Café preto puro em jejum",
      "Ashwagandha 300 mg",
      "Lion's Mane 500 mg",
      "Selênio 100 mcg",
      "Zinco quelado 15 mg",
      "Complexo B ativo"
    ]
  },
  {
    category: "TARDE",
    items: [
      "Almoço leve e funcional",
      "Creatina 3g",
      "Caminhada leve pós-almoço (15 min)",
      "Ômega 3 2-3g",
      "Berberina 500 mg (1ª dose)",
      "Silimarina 300 mg (1ª dose)"
    ]
  },
  {
    category: "NOITE",
    items: [
      "Berberina 500 mg (2ª dose)",
      "Policosanol 10 mg",
      "Silimarina 300 mg (2ª dose)",
      "Curcumina 500 mg",
      "Vitamina D3 + K2",
      "Magnésio Bisglicinato 300 mg",
      "Vinagre de maçã antes da janta",
      "Evitar telas após 21h",
      "Dormir até 22h30"
    ]
  },
  {
    category: "TREINO",
    items: [
      "Academia 1x por dia - configurar dias da semana",
      "Registrar exercícios do dia",
      "Registrar carga, repetições e progresso"
    ]
  },
  {
    category: "MONITORAMENTO DE SAÚDE",
    items: [
      "Subir arquivos de exames para leitura automática (PDF)",
      "Atualizar dados hormonais, vitamínicos e metabólicos"
    ]
  }
];

export default function App() {
  const [checkedItems, setCheckedItems] = useState({});
  const [workoutLog, setWorkoutLog] = useState("");
  const [examFile, setExamFile] = useState(null);
  const [chatMessages, setChatMessages] = useState([]);
  const [newMessage, setNewMessage] = useState("");

  const toggleItem = (category, item) => {
    const key = `${category}-${item}`;
    setCheckedItems({
      ...checkedItems,
      [key]: !checkedItems[key],
    });
  };

  const sendMessage = () => {
    if (newMessage.trim() === "") return;
    const userMsg = { text: newMessage, sender: "user" };
    const updatedMessages = [...chatMessages, userMsg];

    const commandRegex = /tomar (.+?) (?:às|as) (\\d{1,2})h/i;
    const match = newMessage.match(commandRegex);

    if (match) {
      const suplemento = match[1].trim();
      const hora = match[2].padStart(2, "0") + ":00";
      const item = suplemento.charAt(0).toUpperCase() + suplemento.slice(1);

      const novoItem = `Tomar ${item} às ${hora}`;
      const categoria = "MANHÃ";
      const categoriaIndex = checklistItems.findIndex(sec => sec.category === categoria);

      if (categoriaIndex !== -1 && !checklistItems[categoriaIndex].items.includes(novoItem)) {
        checklistItems[categoriaIndex].items.push(novoItem);
      }

      if ('Notification' in window && Notification.permission === 'granted') {
        setTimeout(() => {
          new Notification(`Lembrete: tomar ${item}`, {
            body: `Conforme solicitado, agora é hora de tomar ${item}.`,
          });
        }, 1000);
      }

      updatedMessages.push({
        text: `Entendido! Adicionei "${novoItem}" à sua rotina da manhã e programei um lembrete para ${hora}.`,
        sender: "bot"
      });
    } else {
      updatedMessages.push({ text: "Recebido! Assim que o sistema estiver online, responderei com inteligência.", sender: "bot" });
    }

    setChatMessages(updatedMessages);
    setNewMessage("");
  };

  return (
    <div className="p-4 space-y-6">
      {checklistItems.map((section) => (
        <div key={section.category} className="border p-4 rounded-lg bg-white shadow">
          <h2 className="text-xl font-bold mb-2">{section.category}</h2>
          <ul className="space-y-2">
            {section.items.map((item) => {
              const key = `${section.category}-${item}`;
              return (
                <li key={key} className="flex items-center space-x-2">
                  <input
                    type="checkbox"
                    id={key}
                    checked={checkedItems[key] || false}
                    onChange={() => toggleItem(section.category, item)}
                  />
                  <label htmlFor={key} className="text-base cursor-pointer">{item}</label>
                </li>
              );
            })}
          </ul>

          {section.category === "TREINO" && (
            <div className="mt-4">
              <label htmlFor="workoutLog" className="block font-semibold mb-1">Registro de Exercícios do Dia</label>
              <textarea
                id="workoutLog"
                value={workoutLog}
                onChange={(e) => setWorkoutLog(e.target.value)}
                placeholder="Ex: Supino 4x10 60kg, Agachamento 3x12 80kg..."
                className="w-full border p-2 rounded"
              />
            </div>
          )}

          {section.category === "MONITORAMENTO DE SAÚDE" && (
            <div className="mt-4 space-y-2">
              <label htmlFor="examUpload" className="block font-semibold">Upload de Exames (PDF)</label>
              <input
                type="file"
                id="examUpload"
                accept="application/pdf"
                onChange={(e) => setExamFile(e.target.files[0])}
                className="block"
              />
              {examFile && (
                <p className="text-sm text-gray-600">Arquivo selecionado: {examFile.name}</p>
              )}
            </div>
          )}
        </div>
      ))}

      {/* Chat Assistente */}
      <div className="border p-4 rounded-lg bg-white shadow">
        <h2 className="text-xl font-bold mb-2">Assistente Saldanha</h2>
        <div className="h-48 overflow-y-auto border p-2 mb-2 bg-gray-50 rounded">
          {chatMessages.map((msg, idx) => (
            <div key={idx} className={`text-sm mb-1 ${msg.sender === "user" ? "text-right" : "text-left text-blue-700"}`}>
              {msg.text}
            </div>
          ))}
        </div>
        <div className="flex space-x-2">
          <input
            type="text"
            placeholder="Digite sua dúvida..."
            value={newMessage}
            onChange={(e) => setNewMessage(e.target.value)}
            className="flex-1 border p-2 rounded"
          />
          <button onClick={sendMessage} className="bg-blue-600 text-white px-4 py-2 rounded">Enviar</button>
        </div>
      </div>
    </div>
  );
}
