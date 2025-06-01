# dating-helper
'use client';

import { useState } from 'react';

export default function Home() {
  const [formData, setFormData] = useState({
    name: '',
    age: '',
    job: '',
    hobbies: '',
    personality: '',
  });

  const handleChange = (e: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement>) => {
    setFormData({ ...formData, [e.target.name]: e.target.value });
  };

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    const existing = JSON.parse(localStorage.getItem('girls') || '[]');
    const updated = [...existing, formData];
    localStorage.setItem('girls', JSON.stringify(updated));
    alert('保存成功！');

    setFormData({
      name: '',
      age: '',
      job: '',
      hobbies: '',
      personality: '',
    });
  };

  return (
    <main className="p-4 max-w-xl mx-auto">
      <h1 className="text-2xl font-bold mb-4">✍️信息录入</h1>
      <form className="space-y-4" onSubmit={handleSubmit}>
        <input name="name" value={formData.name} onChange={handleChange} placeholder="姓名" className="w-full border p-2 rounded" />
        <input name="age" value={formData.age} onChange={handleChange} placeholder="年龄" type="number" className="w-full border p-2 rounded" />
        <input name="job" value={formData.job} onChange={handleChange} placeholder="职业" className="w-full border p-2 rounded" />
        <input name="hobbies" value={formData.hobbies} onChange={handleChange} placeholder="兴趣爱好" className="w-full border p-2 rounded" />
        <textarea name="personality" value={formData.personality} onChange={handleChange} placeholder="性格描述" className="w-full border p-2 rounded" rows={3}></textarea>
        <button type="submit" className="bg-blue-500 text-white p-2 rounded w-full">保存</button>
      </form>
    </main>
  );
}
