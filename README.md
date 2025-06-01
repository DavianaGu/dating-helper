# dating-helper'use client';

import { useEffect, useState } from 'react';

interface Girl {
  name: string;
  age: string;
  job: string;
  hobbies: string;
  personality: string;
  score?: number; // 新增字段，好感度，1~10
  aiSummary?: string; // AI总结（后面填）
}

export default function GirlsList() {
  const [girls, setGirls] = useState<Girl[]>([]);
  const [editingIndex, setEditingIndex] = useState<number | null>(null);
  const [editForm, setEditForm] = useState<Girl>({
    name: '',
    age: '',
    job: '',
    hobbies: '',
    personality: '',
    score: undefined,
    aiSummary: '',
  });
  const [loadingAI, setLoadingAI] = useState(false);

  useEffect(() => {
    const stored = localStorage.getItem('girls');
    if (stored) {
      setGirls(JSON.parse(stored));
    }
  }, []);

  const handleDelete = (index: number) => {
    const updated = [...girls];
    updated.splice(index, 1);
    setGirls(updated);
    localStorage.setItem('girls', JSON.stringify(updated));
  };

  const handleEdit = (index: number) => {
    setEditingIndex(index);
    setEditForm(girls[index]);
  };

  const handleEditChange = (e: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement>) => {
    const { name, value } = e.target;
    setEditForm({ ...editForm, [name]: name === 'score' ? Number(value) : value });
  };

  const handleEditSave = () => {
    if (editingIndex === null) return;
    const updated = [...girls];
    updated[editingIndex] = editForm;
    setGirls(updated);
    localStorage.setItem('girls', JSON.stringify(updated));
    setEditingIndex(null);
  };

  // 调用 GPT AI 总结
  const handleAISummarize = async (index: number) => {
    const girl = girls[index];
    if (!girl.personality) {
      alert('请先填写性格描述');
      return;
    }
    setLoadingAI(true);
    try {
      // 调用后端接口触发 GPT，示例写法，后面我们需要做后端API代理
      const res = await fetch('/api/generate-summary', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ personality: girl.personality, hobbies: girl.hobbies }),
      });
      const data = await res.json();
      if (data.summary) {
        const updated = [...girls];
        updated[index].aiSummary = data.summary;
        setGirls(updated);
        localStorage.setItem('girls', JSON.stringify(updated));
      } else {
        alert('AI 生成失败');
      }
    } catch (error) {
      alert('调用 AI 接口出错');
    }
    setLoadingAI(false);
  };

  return (
    <main className="p-4 max-w-3xl mx-auto">
      <h1 className="text-2xl font-bold mb-4">已录入女生列表</h1>
      {girls.length === 0 ? (
        <p>暂无数据。</p>
      ) : (
        <div className="space-y-4">
          {girls.map((girl, index) => (
            <div key={index} className="border rounded p-4 shadow space-y-2">
              {editingIndex === index ? (
                <div className="space-y-2">
                  <input
                    name="name"
                    value={editForm.name}
                    onChange={handleEditChange}
                    placeholder="姓名"
                    className="w-full border p-1 rounded"
                  />
                  <input
                    name="age"
                    value={editForm.age}
                    onChange={handleEditChange}
                    placeholder="年龄"
                    className="w-full border p-1 rounded"
                  />
                  <input
                    name="job"
                    value={editForm.job}
                    onChange={handleEditChange}
                    placeholder="职业"
                    className="w-full border p-1 rounded"
                  />
                  <input
                    name="hobbies"
                    value={editForm.hobbies}
                    onChange={handleEditChange}
                    placeholder="兴趣爱好"
                    className="w-full border p-1 rounded"
                  />
                  <textarea
                    name="personality"
                    value={editForm.personality}
                    onChange={handleEditChange}
                    placeholder="性格描述"
                    className="w-full border p-1 rounded"
                  />
                  <input
                    type="number"
                    name="score"
                    value={editForm.score ?? ''}
                    onChange={handleEditChange}
                    placeholder="好感度（1-10）"
                    min={1}
                    max={10}
                    className="w-full border p-1 rounded"
                  />
                  <div className="flex gap-2">
                    <button onClick={handleEditSave} className="bg-green-500 text-white px-2 py-1 rounded">
                      保存
                    </button>
                    <button onClick={() => setEditingIndex(null)} className="bg-gray-400 text-white px-2 py-1 rounded">
                      取消
                    </button>
                  </div>
                </div>
              ) : (
                <>
                  <p><strong>姓名：</strong>{girl.name}</p>
                  <p><strong>年龄：</strong>{girl.age}</p>
                  <p><strong>职业：</strong>{girl.job}</p>
                  <p><strong>兴趣：</strong>{girl.hobbies}</p>
                  <p><strong>性格：</strong>{girl.personality}</p>
                  <p><strong>好感度：</strong>{girl.score ?? '未评分'}</p>
                  {girl.aiSummary && (
                    <div className="p-2 mt-2 bg-yellow-100 rounded">
                      <strong>AI总结：</strong>
                      <p>{girl.aiSummary}</p>
                    </div>
                  )}
                  <div className="flex gap-2 mt-2">
                    <button onClick={() => handleEdit(index)} className="bg-yellow-500 text-white px-2 py-1 rounded">
                      修改
                    </button>
                    <button onClick={() => handleDelete(index)} className="bg-red-500 text-white px-2 py-1 rounded">
                      删除
                    </button>
                    <button
                      onClick={() => handleAISummarize(index)}
                      disabled={loadingAI}
                      className="bg-blue-600 text-white px-2 py-1 rounded"
                    >
                      {loadingAI ? 'AI生成中...' : 'AI总结'}
                    </button>
                  </div>
                </>
              )}
            </div>
          ))}
        </div>
      )}
    </main>
  );
}
